# ROCK 5B High-Power CAN-Bus & Power HAT ("The Beast")

> 🌍 **Languages:** [English](Readme.md) | [Deutsch](Readme.de.md)

![3D view](img/hat.png)

## Table of Contents


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [ROCK 5B High-Power CAN-Bus & Power HAT ("The Beast")](#rock-5b-high-power-can-bus--power-hat-the-beast)
  - [Table of Contents](#table-of-contents)
  - [About the Project](#about-the-project)
  - [Features & Technical Specifications](#features--technical-specifications)
    - [⚡ Power Distribution (Split-Path Architecture)](#-power-distribution-split-path-architecture)
    - [**🚀 Native CAN-Bus**](#-native-can-bus)
    - [🔌 Poka-Yoke Connectors](#-poka-yoke-connectors)
  - [Pinout](#pinout)
    - [J102 - TOOLHEAD (4-Pin Micro-Fit Square)](#j102---toolhead-4-pin-micro-fit-square)
    - [J103 - SPIDER / MCU (3-Pin Micro-Fit Row)](#j103---spider--mcu-3-pin-micro-fit-row)
    - [J_LCD - MONITOR (4-Pin Micro-Fit Row)](#j_lcd---monitor-4-pin-micro-fit-row)
  - [Status LEDs ("Mouse Cinema")](#status-leds-mouse-cinema)
  - [Software Configuration (Radxa OS / Armbian)](#software-configuration-radxa-os--armbian)
    - [1. Enable Overlay](#1-enable-overlay)
    - [2. Configure Interface](#2-configure-interface)
    - [3. Klipper Configuration](#3-klipper-configuration)
  - [Manufacturing Notes (BOM & PCB)](#manufacturing-notes-bom--pcb)
  - [License](#license)

<!-- /code_chunk_output -->



## About the Project

This HAT was developed to turn the **Radxa Rock 5B** into the ultimate control center for high-performance laser plotters and 3D printers (Klipper). It addresses the specific weaknesses of standard solutions:

1. **Stable Power Supply:** Delivers massive 5.2V/6A for the SBC and peripheral USB-C touchscreens.  
2. **Native CAN Performance:** Uses the integrated CAN controller of the RK3588 instead of slow SPI bridges.  
3. **Thermal Management:** A special "donut" design allows the CPU fan to draw fresh air through the PCB.

## Features & Technical Specifications

### ⚡ Power Distribution (Split-Path Architecture)

The board separates power paths for maximum safety and performance:

* **Input:** 24V DC via reverse-polarity protected **XT30 (High Current)** connector.  
* **High-Power Path (Laser/Toolhead):**  
  * Massive 3-layer copper connection.  
  * Direct 24V passthrough to toolhead (up to 120W+ possible, depending on power supply).  
  * Protected by **SMBJ24A TVS diode** against inductive voltage spikes (Back-EMF).  
* **Logic Path (SBC & Display):**  
  * Protected by **2A Fast** fuse (Littelfuse 0466 series, 63V rating).  
  * **DC/DC Converter:** TI **TPS56637** Synchronous Buck Converter.  
  * Output: **5.2V** (compensates for voltage drop on cables) at up to **6A**.

### **🚀 Native CAN-Bus**

* No USB adapters or SPI chips (MCP2515) required.  
* Uses the native **CAN1 Controller** of the RK3588 (Pins 32/33).  
* **Transceiver:** TI SN65HVD230 (3.3V Logic).  
* **ESD Protection:** NUP2105L diode on data lines.  
* **Termination:** 120Ω termination resistor switchable via solder jumper (JP1).

### 🔌 Poka-Yoke Connectors

Uses **Molex Micro-Fit 3.0** with different pin counts to physically prevent fatal miswiring (e.g., 24V to 5V input).

| Port | Type | Pins | Description |
| :---- | :---- | :---- | :---- |
| **24V IN** | **XT30** | 2 | Main input from power supply. |
| **TOOLHEAD** | **Micro-Fit** | 2x2 (4-Pin) | 24V High-Current + CAN data to laser/print head. |
| **CONTROLLER** | **Micro-Fit** | 1x3 (3-Pin) | Pure data connection to MCU mainboard (e.g., Spider). |
| **MONITOR** | **Micro-Fit** | 1x4 (4-Pin) | 2x 5.2V / 2x GND for powering external displays. |

## Pinout

### J102 - TOOLHEAD (4-Pin Micro-Fit Square)

*This is where the laser or print head is connected.*

1. **24V** (Unfused, High Power)  
2. **GND**  
3. **CAN_L**  
4. **CAN_H**

### J103 - SPIDER / MCU (3-Pin Micro-Fit Row)

*Connection to mainboard. Galvanically isolated from 24V.*

1. **GND**  
2. **CAN_L**  
3. **CAN_H**

### J_LCD - MONITOR (4-Pin Micro-Fit Row)

*Power supply for USB-C monitors or HDMI displays.*

1. **+5.2V**  
2. **+5.2V**  
3. **GND**  
4. **GND**

## Status LEDs ("Mouse Cinema")

The board features two labeled LEDs for quick diagnosis:

* 24V input present (fuse intact).  
* 5.2V logic voltage stable (Power Good signal from TPS56637).  

## Software Configuration (Radxa OS / Armbian)

Since the native CAN controller is used, setup is extremely simple.

### 1. Enable Overlay

Add the overlay for the CAN1 controller.  
Via rsetup:  
`Hardware -> Overlays -> Enable CAN1-M1 on GPIO3 (Pins 32/33)`.  
Manually (`/boot/extlinux/extlinux.conf`):  
Add `rk3588-can1-m1` to the `fdtoverlays` line.

### 2. Configure Interface

Create `/etc/network/interfaces.d/can0` for autostart:  
```
allow-hotplug can0 
iface can0 can static  
    bitrate 500000  
    up ip link set $IFACE txqueuelen 1024
```

### 3. Klipper Configuration

In printer.cfg:
```
[mcu]  
canbus_uuid: <your_uuid>  
# No more "serial:" entries!
```

## Manufacturing Notes (BOM & PCB)

* **PCB Specs:** 4-Layer (Signal / GND / Power / Signal), 1oz copper.  
* **GPIO Header:** A **Stacking Header (Extra Tall, min. 11mm spacer height)** is absolutely required to provide clearance from the CPU cooler.  
* **Soldering:** Due to massive ground and power planes, a preheater plate (printer heated bed) or preheating in an oven (100°C) is strongly recommended.

**Critical Components (LCSC):**

* Buck Converter: **TPS56637RPAR**  
* Inductor: **MDA1350-2R2M** (2.2µH, Isat > 15A, Shielded)  
* Transceiver: **SN65HVD230**  
* Fuse: **Littelfuse 0466002.NRHF** (2A, **63V Rating!**)

Disclaimer:  
This design works with high currents and voltages. Use at your own risk. Make sure cable cross-sections are rated for the load of your laser/hotend.

## License

This project is licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License** (CC BY-NC-SA 4.0).

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

This means you are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made
- **NonCommercial** — You may not use the material for commercial purposes
- **ShareAlike** — If you remix, transform, or build upon the material, you must distribute your contributions under the same license

For the full license text, visit: https://creativecommons.org/licenses/by-nc-sa/4.0/

---

> **Note:** The original documentation is in German ([Readme.de.md](Readme.de.md)). This English version is a translation for international users.