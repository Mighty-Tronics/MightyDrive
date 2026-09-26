# Pump Interface — Mighty Tronics®

A four-channel power interface for driving DC pumps from an external microcontroller. This repository documents the design, manufacture, and progressive validation of a hardware prototype.

> **Status: prototype under development.** The specifications below are design targets, not qualified performance claims. Abnormal control signals have been observed on some channels of the prototype. Verify each channel before connecting pumps or applying the pump supply voltage.

## Design targets

| Parameter | Target |
| --- | --- |
| Channels | 4 independently driven pumps |
| Pump supply | 5–36 V DC, externally supplied |
| Continuous current (revised target) | Up to 3 A per channel; 12 A total, subject to validation |
| Startup current | To be characterized with the selected pumps and channel protection |
| Control | External PWM at 20–31 kHz; 3.3 V or 5 V MCU logic levels |
| PCB | Four layers; manual assembly; prototype manufactured by Eurocircuits |

The board provides the control interface and power switching. The microcontroller, firmware, DC supply, and pumps are external. The supply, connectors, cables, traces, and protection devices must all be rated for the actual installation.

The original project charter specified 4 A per channel (16 A total). The provisional target is now 3 A per channel (12 A total) following selection of a 15SQ100 diode. Its 15 A average forward-current rating applies under the manufacturer's specified thermal conditions; **it does not by itself qualify the board for 12 A**. Check the diode's location and actual current path in the manufactured revision, its temperature, startup currents, and the ratings of all other components. A freewheel diode, for example, does not necessarily carry the sum of the supply currents.

## Architecture and development process

The design addresses power distribution, PWM input conditioning, four MOSFET switching stages, and protection against transients from inductive loads. The Mighty Tronics development process covers requirements and validation planning (VS1), architecture and sizing (VS2), schematic design, simulation, PCB layout, manufacturing, and testing.

The supplied legacy KiCad schematic is dated 21 August 2025 and marked revision V1.0. It includes IRLZ44N MOSFETs, but does not identify a 15SQ100. It may not match the manufactured board or its latest bill of materials. Always match schematics, PCB files, assembly data, and test reports to the same hardware revision.

## Validation status

- PWM control is still being debugged. Measurements on the prototype show abnormal levels around the buffer on some channels; the root cause has not been confirmed.
- Functional and thermal testing at 36 V / 3 A per channel, including simultaneous operation of all four channels, remains to be documented.
- EMC pre-compliance tests remain to be documented. No CE conformity claim is made for this prototype.

### Planned test sequence

1. Inspect the unpowered assembly, component orientation, continuity, and possible shorts.
2. Power only the logic with a current-limited supply. Check the PWM inputs, buffer outputs, and MOSFET gates on all four channels.
3. Test each channel with a suitable load and a current-limited pump supply, increasing voltage and current gradually.
4. Test at 36 V / 3 A with PWM duty cycles of 25%, 50%, 80%, and 100%, followed by variable PWM, for one hour per condition.
5. Repeat with all four channels active. Record critical component temperatures (target below 90 °C), waveforms, and any faults.
6. Document transient, reverse-polarity, and EMC pre-compliance tests separately, including their acceptance criteria.

## Working with the repository

Open the design in KiCad and check the revision of the schematic, PCB, bill of materials, and manufacturing files before assembly. The VS1 and VS2 documents describe requirements and engineering decisions; they do not replace measurements on built hardware. Add updated design files, BOM, Gerber/Excellon outputs, master drawing, calculations, simulations, and test reports as they become available and verified.

Firmware development, enclosure selection, final regulatory assessment, and high-volume production are outside the current project phase.

## License

| Project material | License |
| --- | --- |
| Hardware design (schematic, PCB, fabrication files and BOM) | **CERN-OHL-S-2.0** |
| Original project documentation | **CERN-OHL-S-2.0** |
| Software / firmware | **None included in this release** |

The full terms of the CERN Open Hardware Licence Version 2 - Strongly Reciprocal are in [`LICENSE`](LICENSE). Modified versions of the hardware design are subject to its reciprocal terms. Add the official license text to that file before publishing this release. If software is added later, identify its license separately.

Third-party data sheets, manufacturer files, and other external materials retain their respective owners' rights. They are not licensed under CERN-OHL-S-2.0 by their inclusion in this repository.
