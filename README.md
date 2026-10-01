# AgroIQ Reference Architecture

A low cost and interoperable architecture for soil condition sensing and irrigation decision support, engineered to the security requirements of a designated critical infrastructure sector. Designed and documented by Carlos Andrés Yoncón Changkuón since 2019.

## Contents

- `AgroIQ-Reference-Architecture-Technical-Record-v12.pdf`: the technical record. It documents the four layers (an industrial Modbus RTU soil probe, an ESP32 node on LoRaWAN 1.0.3 in US915, a self hosted ChirpStack network server, and time series storage with a dashboard), the design decisions and the alternatives rejected, the bill of materials, the security controls with an assessment against NIST IR 8259A, the requirements for deploying the architecture, and a register of every open item and limitation.
- `payload-specification.md`: the uplink payload specification (Appendix A of the record), complete enough to write a decoder in any language.

Source code and the configuration of the development instance are not part of this publication.

## Archived version

Version 12 is archived at https://doi.org/10.5281/zenodo.23087515. Cite that version:

> Yoncón Changkuón, C. A. (2026). *AgroIQ Reference Architecture: Technical Record*, version 12. https://doi.org/10.5281/zenodo.23087515

## License

Everything in this repository is released under the Creative Commons Attribution 4.0 International License (CC BY 4.0). Anyone may build, adapt and deploy the architecture, and publish their own version. Attribution is the only condition. The scope of publication is stated at https://agroiq.metricas.net/license/.

Contact: cyoncon@metricas.net
