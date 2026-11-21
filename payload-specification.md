# AgroIQ uplink payload specification

This is Appendix A of *AgroIQ Reference Architecture: Technical Record*, version 13 (https://doi.org/10.5281/zenodo.23646524), reproduced here so it can be read and linked on its own. Section references point to that record. Released under the Creative Commons Attribution 4.0 International License.

**Correction from version 12.** The copy of this file published with version 12 gave the wrong conversions for `soil_temp` and `soil_moist` (a division by ten, and a division by ten with a minus forty offset) and a worked example that did not match the frames the nodes send. A decoder written from that copy returns wrong values. The conversions in A.3 and the example in A.6 below are the ones my decoder applies.

It documents the AgroIQ uplink frame completely enough that an independent implementer can write a decoder in any language without access to mine.

## A.1 Frame

A single uplink of **42 bytes**, big endian throughout, at fixed offsets. The frame length does not vary with the sensors fitted. Byte 17 carries a capability mask recording which sensors produced a reading; a field whose mask bit is clear still occupies its bytes in the frame, and is simply left out of the decoded record.

## A.2 Fields decoded unconditionally

| Bytes    | Field       | Type   | Conversion                                       | Unit              |
|----------|-------------|--------|--------------------------------------------------|-------------------|
| 0 to 2   | `latitude`  | uint24 | `(raw / 16777215 × 180) − 90`, 6 dp              | degrees           |
| 3 to 5   | `longitude` | uint24 | `(raw / 16777215 × 360) − 180`, 6 dp             | degrees           |
| 6 to 7   | `altitude`  | int16  | bit 7 of byte 6 is the sign; if set, sign extend | meters            |
| 8        | `hdop`      | uint8  | `raw / 10`                                       | none              |
| 9        | `sats`      | uint8  | as read                                          | count             |
| 17       | `capab`     | uint8  | as read                                          | bit mask, see A.4 |
| 18 to 19 | `voltage`   | uint16 | `raw / 1000`                                     | volts             |

## A.3 Fields gated by the capability mask

| Mask condition                | Bytes                          | Field           | Conversion                           | Unit  |
|-------------------------------|--------------------------------|-----------------|--------------------------------------|-------|
| bit 1                         | 12 to 13                       | `temp`          | `raw / 100`                          | °C    |
| bit 1                         | 14 to 15                       | `hum`           | `raw / 100`                          | %     |
| bit 2                         | 10 to 11                       | `soil`          | `((raw − 1200) × −1/30) + 100`, 2 dp | %     |
| bit 3                         | 16                             | `relay`         | as read                              | state |
| bit 4                         | 20 to 21                       | `temp2`         | `raw / 100`                          | °C    |
| bit 4                         | 22 to 23                       | `hum2`          | `raw / 100`                          | %     |
| bit 4                         | 24 to 26                       | `press`         | uint24, as read                      | none  |
| bit 5                         | 27                             | `rain`          | as read                              | none  |
| bit 6 set **and** bit 7 clear | 28 to 29 / 30 to 31 / 32 to 33 | `N` / `P` / `K` | as read                              | mg/kg |
| bit 7 set **and** bit 6 clear | 34 to 35                       | `ph`            | as read, unscaled                    | raw   |
| bit 7 set **and** bit 6 set   | 36 to 37                       | `ec`            | as read, unscaled                    | raw   |
| bits 5, 6 and 7 all set       | 28 to 33                       | `N` / `P` / `K` | as read                              | mg/kg |
| bits 5, 6 and 7 all set       | 34 to 35                       | `ph`            | as read, unscaled                    | raw   |
| bits 5, 6 and 7 all set       | 36 to 37                       | `ec`            | as read, unscaled                    | raw   |
| bits 5, 6 and 7 all set       | 38 to 39                       | `soil_temp`     | `raw / 1000`, 2 dp                   | °C    |
| bits 5, 6 and 7 all set       | 40 to 41                       | `soil_moist`    | `raw / 1000`, 2 dp                   | % VWC |

The `soil` field at bytes 10 to 11 is the legacy channel for the capacitive sensor described in section 3.1, the one that corroded. It is retained in the layout for frames from earlier builds and no current node populates it.

The firmware writes the probe’s temperature and moisture registers, which the probe gives in tenths, multiplied by one hundred into these two 16-bit fields, and the decoder divides by one thousand. Version 12 gave a division by ten for temperature and a division by ten with a minus forty offset for moisture. Those conversions belonged to an earlier decoder and do not match the frames the nodes send.

## A.4 Capability mask, byte 17

| Bit | Meaning                                                                    |
|-----|----------------------------------------------------------------------------|
| 0   | set on every frame by the firmware (GPS, always 1); the decoder ignores it |
| 1   | air temperature and humidity present                                       |
| 2   | legacy capacitive soil channel present                                     |
| 3   | relay state present                                                        |
| 4   | barometric sensor present: temperature, humidity and pressure              |
| 5   | rain sensor present, **and** part of the seven-parameter selector          |
| 6   | nutrient channels present, **and** part of the seven-parameter selector    |
| 7   | pH and conductivity selector, **and** part of the seven-parameter selector |

The seven-parameter probe described in section 3.1 is signaled by bits 5, 6 and 7 set together.

## A.5 Known defects in this format

The format grew a sensor at a time, and these are the consequences. An implementer needs them, and section 9 carries them as open items.

**Bit 5 is overloaded.** It signals both that a rain sensor is fitted and that the frame is a full seven-parameter frame. A node cannot report one condition without asserting the other, so a seven-parameter node necessarily claims a rain sensor and byte 27 is decoded as a rain reading whether or not one exists.

**No length validation.** The format defines no minimum length. Until 16 October 2026 my decoder checked none, so a truncated uplink produced arithmetic on undefined bytes and yielded NaN. It now rejects a frame that is not 42 bytes long, and a decoder written from this specification needs the same check.

**pH and conductivity are unscaled.** Both are stored as the raw 16-bit values. Neither is in a physical unit, and the scale factor the probe applies is not documented by its manufacturer in a form I have been able to verify. Section 9 records that these channels are not relied on by any function, which is why the omission has had no operational consequence, and it is a defect in the format regardless.

**Moisture above 65.535 percent wraps.** The field holds the moisture in thousandths of a percent in 16 bits, so a reading above 65.535 percent, which a saturated soil can give, wraps to a value near zero.

**A soil temperature below zero decodes as a high value.** The field is read as unsigned, so a temperature between 0 and −10 °C decodes as a value between about 65.5 and 55.5 °C. Both this value and a wrapped moisture pass the decoder’s range check, and the revised frame format, an open item in section 9, corrects them. Both wraps occurred in the check of 5 November 2026 (section 3.4): water, which the probe reads as 100 percent, was stored as 34.46 percent, a reading of 84.2 percent as 18.66, and −1.4 °C as 64.1 °C.

**The format has no room to grow.** Bit 0 of the mask carries no information, since the firmware sets it on every frame, and every byte of the 42-byte frame is allocated, three mask bits carry combined meanings, and a further sensor class cannot be added without extending the frame.

**No format version field.** The frame carries no version identifier, so a receiver cannot tell which revision of this specification a given uplink was produced under. The `soil_temp` and `soil_moist` conversions in A.3 changed during development, and a frame from before that change decodes to the wrong values with no indication.

## A.6 Worked example

A node in service, with the seven-parameter probe (bits 5, 6 and 7) and bit 0 set, reports `capab` = 225 (0xE1). Given bytes 38 to 39 = `0x5FB4` and bytes 40 to 41 = `0x6C98`, the decoder gives a soil temperature of 24.50 °C (24,500 divided by 1,000) and a volumetric water content of 27.80 percent (27,800 divided by 1,000).
