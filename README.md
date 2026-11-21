# AgroIQ Reference Architecture

A low-cost and interoperable architecture for soil condition sensing and irrigation decision support, engineered to the security requirements of a designated critical infrastructure sector. Designed and documented by Carlos Andrés Yoncón Changkuón since 2019.

## Contents

- `AgroIQ-Reference-Architecture-Technical-Record-v13.pdf`: the technical record, version 13. It documents the four layers (an industrial Modbus RTU soil probe, an ESP32 node on LoRaWAN 1.0.3 in US915, a self-hosted ChirpStack network server, and time-series storage with a dashboard), the design decisions and the alternatives rejected, the bill of materials, the security controls with an assessment against NIST IR 8259A, the requirements for deploying the architecture, and a register of every open item and limitation. Appendix B sets out a draft method for irrigation thresholds drawn from published land-grant extension and NRCS guidance, and Appendix C the operating record of a thirty-one-day run.
- `payload-specification.md`: the uplink payload specification (Appendix A of the record), complete enough to write a decoder in any language. It replaces the copy published with version 12, whose conversions for soil temperature and soil moisture were wrong.
- `AgroIQ-Reference-Architecture-Technical-Record-v12.pdf`: the previous version, kept for reference. The section "Changes from version 12" in version 13 lists what was corrected and what was added.
- `SECURITY.md`: how to report a vulnerability.

Source code and the configuration of the development instance are not part of this publication.

## Archived versions

Version 13 is archived at https://doi.org/10.5281/zenodo.23646524. Cite that version:

> Yoncón Changkuón, C. A. (2026). *AgroIQ Reference Architecture: Technical Record*, version 13. https://doi.org/10.5281/zenodo.23646524

Version 12 remains archived at https://doi.org/10.5281/zenodo.23087515.

## Related writing

- "Seven years building a soil sensing network that a small farm could actually afford", Hackster.io, 5 October 2026, updated in November 2026: https://www.hackster.io/carlos-yoncon/soil-sensing-network-for-small-farms-d77ec6. What broke during development, what the node costs, and what is still open.

## License

Everything in this repository is released under the Creative Commons Attribution 4.0 International License (CC BY 4.0). Anyone may build, adapt and deploy the architecture, and publish their own version. Attribution is the only condition. The scope of publication is stated at https://agroiq.metricas.net/license/.

Contact: cyoncon@metricas.net
