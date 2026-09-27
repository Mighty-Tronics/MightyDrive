# MT Drive — Four-Channel PWM Pump Interface

MT Drive is an open-hardware power interface for controlling up to four brushed DC pumps or comparable inductive DC loads from an external microcontroller.

Each channel accepts an independent PWM command and switches the load through a protected low-side MOSFET stage. The microcontroller, firmware, regulated +5 V logic supply, motor supply and pumps remain external to the board.

> **Hardware status:** prototype revision V2.0, manufactured by PCBWay and under qualification.  
> **Published electrical rating:** 2 A continuous per channel, 8 A continuous total.  
> **Extended operation:** up to 3 A per channel may be characterized, but is not guaranteed for this hardware revision.

## Main specifications

| Characteristic | Published specification |
| --- | --- |
| Number of channels | 4 independent outputs |
| Motor supply | 5–36 V DC, externally supplied |
| Continuous current | 2 A maximum per channel |
| Aggregate continuous current | 8 A maximum |
| Extended characterization | Up to 3 A per channel / 12 A total, not guaranteed |
| PWM frequency | 31.5 kHz nominal |
| PWM input levels | 3.3 V or 5 V logic |
| Logic supply | Regulated +5 V external supply |
| Switching topology | N-channel MOSFET, low-side switching |
| Intended ambient range | −10 °C to +50 °C |
| Galvanic isolation | None; logic and motor power use a common ground |
| Hardware reference | Pump Interface schematic V2.0, dated 24 April 2026 |

At 36 V and the published current rating, the board can deliver up to 72 W per load and 288 W to four loads. These values are load power, not PCB dissipation.

## Functional operation

### Power entry

The motor supply enters through `J1`:

- `PWR`: positive motor supply, 5–36 V DC;
- `GND`: common power and logic reference.

The common input stage includes:

- `D5` 15SQ100 series Schottky diode for reverse-polarity protection;
- `D6` 1.5KE43A TVS diode for supply transient suppression;
- `C1` 1000 µF, `C2` 10 µF and `C3` 100 nF for rail decoupling;
- `TP1` and `TP2` for protected `Vin+` and ground measurements.

### PWM command path

Each PWM input passes through a 1 kΩ series resistor and a 100 kΩ pull-down before entering one section of the `74AHCT125` buffer. The buffer is powered from +5 V and adapts a 3.3 V or 5 V controller signal to the 5 V MOSFET gate-control network.

The `74AHCT125` has been bench-tested for this level-adaptation function. It carries gate-drive current only and does not determine the motor-current rating.

The four buffer outputs are permanently enabled because their active-low `/OE` inputs are tied to ground. Gate pull-down resistors keep the MOSFETs off when the controller is disconnected or its outputs are high-impedance.

### Load switching

When a PWM command is high, the corresponding `IRLZ44N` MOSFET turns on. Current flows from `Vin+`, through the channel PTC, through the external load connected between `Mx+` and `Mx−`, and through the MOSFET to ground.

When the MOSFET turns off, the channel flyback diode maintains a local path for the inductive load current. A 22 Ω + 1 nF series RC snubber damps high-frequency ringing across each motor output.

## Connections

### J2 — Logic and PWM

| Pin | Signal | Description |
| ---: | --- | --- |
| 1 | +5 V | External regulated logic supply |
| 2 | GND | Common controller and power-stage reference |
| 3 | PWM1 | Channel 1 command |
| 4 | PWM2 | Channel 2 command |
| 5 | PWM3 | Channel 3 command |
| 6 | PWM4 | Channel 4 command |

### J3 — Pump outputs

| Pin | Signal | Pin | Signal |
| ---: | --- | ---: | --- |
| 1 | M3+ | 2 | M3− |
| 3 | M4+ | 4 | M4− |
| 5 | M2+ | 6 | M2− |
| 7 | M1+ | 8 | M1− |

### Test points

| Test point | Signal |
| --- | --- |
| TP1 | Protected motor rail `Vin+` |
| TP2 | GND |
| TP3 | Channel 1 gate, `Qg1` |
| TP10 | Channel 2 gate, `Qg2` |
| TP11 | Channel 3 gate, `Qg3` |
| TP12 | Channel 4 gate, `Qg4` |

## Protection and reliability architecture

Each channel contains:

- one `MF-R500` resettable PTC in the positive load feed;
- one `15SQ100` flyback diode;
- local 22 µF + 100 nF supply decoupling;
- one 22 Ω / 0.5 W + 1 nF series RC snubber;
- a 10 kΩ MOSFET gate pull-down;
- a 1N4148 gate clamp and gate-conditioning capacitors.

The protections have different roles and do not replace one another. The PTC limits sustained fault energy but is not a precision current limiter. The flyback diode controls inductive current, the snubber damps local ringing, and the TVS limits disturbances reaching the common supply rail.

The 55 V `IRLZ44N` has 19 V of absolute drain-voltage headroom at a 36 V supply. The internal qualification target is a repetitive `VDS` peak no greater than 45 V under the published load, worst switching condition and longest intended motor cable.

## Current and thermal basis

The 2 A/channel rating is based on the complete current path rather than on the headline rating of a single semiconductor.

- Four channels at 2 A give an 8 A continuous common-path current.
- Using the 15SQ100 maximum forward-voltage screening value of 0.82 V gives a conservative `D5` loss bound of 6.56 W at 8 A. Actual forward drop and case temperature must be measured.
- The `MF-R500` hold current falls with ambient temperature; 2 A retains useful margin up to the declared +50 °C design ambient.
- MOSFET conduction and switching losses are verified at 31.5 kHz from measured gate and drain waveforms.
- Power-component case temperatures are checked against the 90 °C internal qualification target.

Operation above 2 A per channel, up to 3 A, is an engineering-characterization region only. It depends on duty cycle, pump start and stall current, ambient temperature, airflow, wiring, connector losses and simultaneous channel loading.

## Integration requirements

1. Connect the controller ground to `J2 GND` before applying PWM signals.
2. Apply a regulated +5 V supply to `J2`; the board does not generate this voltage.
3. Use a current-limited motor supply for first power-up.
4. Size the supply, connectors and cable loop for 8 A continuous total current plus pump start-current transients.
5. Keep PWM and ground wiring short and route motor cables away from sensitive logic wiring.
6. Do not exceed 2 A continuous on any channel or 8 A continuous in total within the published specification.
7. Verify each pump's start and stall current before qualification in the final system.

The board does not regulate motor voltage, provide active electronic current limiting or provide galvanic isolation. It is a component for integration, not a complete protected end product.

## Validation plan

The production-representative board is verified using the following sequence:

1. Inspect component orientation, solder joints and unpowered resistance paths.
2. Apply logic power only and verify all four PWM inputs, buffer outputs and MOSFET gates at 3.3 V and 5 V input levels.
3. Test each channel with a current-limited resistive load before connecting pumps.
4. Verify operation at 31.5 kHz and 10%, 50% and 90% duty cycle.
5. Qualify one channel at 36 V / 2 A over the full duty-cycle range.
6. Qualify four simultaneous channels at 36 V / 2 A per channel after thermal stabilization.
7. Measure MOSFET `VDS` peaks and ringing with the longest intended cable.
8. Record start, stall, reverse-polarity, endurance and high-ambient behaviour.
9. Characterize operation progressively above 2 A and up to 3 A separately, without changing the published rating.
10. Complete EMC pre-compliance tests using the final PCB, pumps, supply and cabling.

No CE conformity claim is made solely from this prototype or this repository. Regulatory assessment applies to the final integrated product and its intended use.

## Repository contents

The release repository should keep the editable design sources and the generated manufacturing outputs clearly separated:

```text
.
├── Documentation/  #Includes BOM (ODS and CSV format), schmeatics (PNGformat), drill maps (PDF format), gerbers and Excellon files for PCB manuifacturers (Zip file), Master drawing (PDF format) and Functional and Engineering notes (PDF format)
├── Kicad_Files/    #Includes Includes all Kicad files to view or edit the shcematic and pcb.
├── README.md       #Quick project description
├── LICENSE.md      #License description

```

The native KiCad files are the authoritative editable design sources. Rendered schematics, PDFs and manufacturing files are provided for convenient review and production, but do not replace the native source files.

Every hardware release must identify its PCB revision or release date and match it to an immutable repository tag, BOM, schematic, layout and validation evidence.
Master Drawing (https://github.com/Mighty-Tronics/MightyDrive/blob/main/Documentation/Pump_Interface_MasterDrawing_Rev_A.pdf)

## Open hardware status

The design is released under the **CERN Open Hardware Licence Version 2 — Strongly Reciprocal (`CERN-OHL-S-2.0`)**. The complete license text must be available in [`LICENSE.md`](LICENSE.md).

| Project material | License |
| --- | --- |
| Hardware design, schematic, PCB, fabrication files and BOM | CERN-OHL-S-2.0 |
| Original project documentation | CERN-OHL-S-2.0 |
| Software or firmware | None included in this release |

Third-party data sheets and manufacturer files retain their owners' respective rights and are not relicensed by their inclusion or citation.

The project is being documented for an open-hardware release. 
- [CERN Open Hardware Licence Version 2](https://ohwr.org/cern_ohl_s_v2.txt)

## Documentation

The detailed functional description, engineering calculations, component-level implementation and qualification criteria are provided in the project documentation under Doucmentation/.

## Maintainer
[info@mightytronics.eu](mailto:info@mightytronics.eu)
