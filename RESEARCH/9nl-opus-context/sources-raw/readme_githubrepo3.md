# Repo 3: TeensyFOC-Carrier

> **Link:** https://github.com/tthom289/TeensyFOC-Carrier
> **Licencia:** MIT | **Lenguaje:** KiCad (PCB design) | **Stars:** 15 | **Forks:** 0
> **Contributors:** tthom289 (Tyler)

## Descripcion (About)

Teensy 4.0 + SimpleFOC carrier board for brushless motor control in a rotating LiDAR SLAM platform. Designed for the Subterranean Systems VLP-16 scanner — integrates AS5600 magnetic encoder, slip ring passthrough, and GT2 belt drive motor driver circuitry on a compact custom PCB.

## Overview

This board eliminates the wiring mess of a breadboarded SimpleFOC setup by consolidating the Teensy 4.0, motor driver interface, and encoder connections onto a single PCB. Designed specifically for the rotating LiDAR scanner in the Subterranean Systems cave mapping platform.

## Key features

- Teensy 4.0 footprint with all relevant I/O broken out
- SimpleFOC-compatible motor driver header
- AS5600 magnetic encoder interface (I2C)
- JST-XHP connectors for motor phase and power leads
- Slip ring wiring passthrough support
- Designed in KiCad

## Estructura del repo

| Archivo | Descripcion |
|---|---|
| `TeensyFOC.kicad_sch` | Esquematico KiCad |
| `TeensyFOC.kicad_pcb` | Layout PCB |
| `TeensyFOC.kicad_pro` | Proyecto KiCad |
| `TeensyFOC.kicad_prl` | Preferencias locales KiCad |
| `TeensyFOC.step` | Modelo 3D |
| `TeensyFOC.zip` | Gerbers para fabricacion |
| `TeensyFOC.csv` | Netlist CSV |
| `TeensyFOC_bom.csv` | Bill of Materials |
| `TeensyFOC_designators.csv` | Designadores de componentes |
| `TeensyFOC_positions.csv` | Posiciones pick-and-place |
| `TeensyMotorCTRL.zip` | Zip adicional (motor control) |
| `fabrication-toolkit-options.json` | Opciones del toolkit de fabricacion |
| `fp-info-cache` | Cache de footprints KiCad |
| `netlist.ipc` | Netlist IPC |

## Bill of Materials (Connectors)

| Component | Link |
|---|---|
| Single Row 2.54mm Headers | Amazon |
| JST-XHP Connector Kit | Amazon |

## Related

- [Subterranean Systems YouTube](https://www.youtube.com/@SubterraneanSystems) — build videos for this platform
- [SimpleFOC](https://simplefoc.com/) — motor control library
