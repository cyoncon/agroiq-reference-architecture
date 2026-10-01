# AgroIQ uplink payload specification

This is Appendix A of *AgroIQ Reference Architecture: Technical Record*, version 12 (https://doi.org/10.5281/zenodo.23087515), reproduced here so it can be read and linked on its own. Section references point to that record. Released under the Creative Commons Attribution 4.0 International License.

It documents the AgroIQ uplink frame completely enough that an independent implementer can write a decoder in any language without access to mine.

## A.1 Frame

A single uplink of **42 bytes**, big endian throughout, at fixed offsets. The frame length does not vary with the sensors fitted. Byte 17 carries a capability mask recording which sensors produced a reading; a field whose mask bit is clear still occupies its bytes in the frame, and is simply left out of the decoded record.

## A.2 Fields decoded unconditionally

| Bytes | Field | Type | Conversion | Unit |
|---|---|---|---|---|
| 0–2 | `latitude` | uint24 | `(raw / 16777215 × 180) − 90`, 6 dp | degrees |
| 3–5 | `longitude` | uint24 | `(raw / 16777215 × 360) − 180`, 6 dp | degrees |
| 6–7 | `altitude` | int16 | bit 7 of byte 6 is the sign; if set, sign extend | meters |
| 8 | `hdop` | uint8 | `raw / 10` | none |
| 9 | `sats` | uint8 | as read | count |
| 17 | `capab` | uint8 | as read | bit mask, see A.4 |
| 18–19 | `voltage` | uint16 | `raw / 1000` | volts |

## A.3 Fields gated by the capability mask

| Mask condition | Bytes | Field | Conversion | Unit |
|---|---|---|---|---|
| bit 1 | 12–13 | `temp` | `raw / 100` | °C |
| bit 1 | 14–15 | `hum` | `raw / 100` | % |
| bit 2 | 10–11 | `soil` | `((raw − 1200) × −1/30) + 100`, 2 dp | % |
| bit 3 | 16 | `relay` | as read | state |
| bit 4 | 20–21 | `temp2` | `raw / 100` | °C |
| bit 4 | 22–23 | `hum2` | `raw / 100` | % |
| bit 4 | 24–26 | `press` | uint24, as read | none |
| bit 5 | 27 | `rain` | as read | none |
| bit 6 set **and** bit 7 clear | 28–29 / 30–31 / 32–33 | `N` / `P` / `K` | as read | mg/kg |
| bit 7 set **and** bit 6 clear | 34–35 | `ph` | as read, unscaled | raw |
| bit 7 set **and** bit 6 set | 36–37 | `ec` | as read, unscaled | raw |
| bits 5, 6 and 7 all set | 28–33 | `N` / `P` / `K` | as read | mg/kg |
| bits 5, 6 and 7 all set | 34–35 | `ph` | as read, unscaled | raw |
| bits 5, 6 and 7 all set | 36–37 | `ec` | as read, unscaled | raw |
| bits 5, 6 and 7 all set | 38–39 | `soil_temp` | `raw / 10`, 2 dp | °C |
| bits 5, 6 and 7 all set | 40–41 | `soil_moist` | `(raw / 10) − 40` | % VWC |

The `soil` field at bytes 10–11 is the legacy channel for the capacitive sensor described in section 3.1, the one that corroded. It is retained in the layout for frames from earlier builds and no current node populates it.

The `soil_temp` and `soil_moist` conversions carry the offsets applied after the calibration described in section 3.4: a divide by ten on temperature and a divide by ten with a minus forty offset on moisture.

## A.4 Capability mask, byte 17

| Bit | Meaning |
|---|---|
| 0 | unused |
| 1 | air temperature and humidity present |
| 2 | legacy capacitive soil channel present |
| 3 | relay state present |
| 4 | barometric sensor present: temperature, humidity and pressure |
| 5 | rain sensor present, **and** part of the seven parameter selector |
| 6 | nutrient channels present, **and** part of the seven parameter selector |
| 7 | pH and conductivity selector, **and** part of the seven parameter selector |

The seven parameter probe described in section 3.1 is signaled by bits 5, 6 and 7 set together.

## A.5 Known defects in this format

The format grew a sensor at a time, and these are the consequences. An implementer needs them, and section 9 carries them as open items.

**Bit 5 is overloaded.** It signals both that a rain sensor is fitted and that the frame is a full seven parameter frame. A node cannot report one condition without asserting the other, so a seven parameter node necessarily claims a rain sensor and byte 27 is decoded as a rain reading whether or not one exists.

**No length validation.** The format defines no minimum length and my decoder checks none. A truncated uplink produces arithmetic on undefined bytes and yields NaN, and nothing downstream distinguishes that from a reading. Section 5.7 records the same gap as the absence of range and rate of change validation.

**pH and conductivity are unscaled.** Both are stored as the raw sixteen bit values. Neither is in a physical unit, and the scale factor the probe applies is not documented by its manufacturer in a form I have been able to verify. Section 9 records that these channels are not relied on by any function, which is why the omission has had no operational consequence, and it is a defect in the format regardless.

**The format has no room to grow.** Bit 0 of the mask is unused, but every byte of the 42 byte frame is allocated, three mask bits carry combined meanings, and a further sensor class cannot be added without extending the frame.

**No format version field.** The frame carries no version identifier, so a receiver cannot tell which revision of this specification a given uplink was produced under. The `soil_temp` and `soil_moist` offsets in A.3 changed during development, and a frame from before that change decodes to the wrong values with no indication.

## A.6 Worked example

A seven parameter node with bits 5, 6 and 7 set reports `capab` = 224 plus whichever of bits 1 to 4 apply. Given bytes 38–39 = `0x00F5` and bytes 40–41 = `0x02A6`, the node reports a soil temperature of 24.50 °C and a volumetric water content of 27.8 percent.
