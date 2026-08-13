<div align="center">

```text
  ████████╗██████╗ ██╗██████╗ ███████╗███╗   ██╗████████╗
  ╚══██╔══╝██╔══██╗██║██╔══██╗██╔════╝████╗  ██║╚══██╔══╝
     ██║   ██████╔╝██║██║  ██║█████╗  ██╔██╗ ██║   ██║   
     ██║   ██╔══██╗██║██║  ██║██╔══╝  ██║╚██╗██║   ██║   
     ██║   ██║  ██║██║██████╔╝███████╗██║ ╚████║   ██║   
     ╚═╝   ╚═╝  ╚═╝╚═╝╚═════╝ ╚══════╝╚═╝  ╚═══╝   ╚═╝


# TRIDENT 328P

<p align="center">
  <strong>Sensorless 3-Phase BLDC Electronic Speed Controller</strong>
</p>

<p align="center">
  Open Hardware • ATmega328P • Sensorless 6-Step Commutation
</p>

<p align="center">

![MCU](https://img.shields.io/badge/MCU-ATmega328P--AU-0A7EA4?style=for-the-badge)
![ESC](https://img.shields.io/badge/ESC-3--Phase%20BLDC-1F6FEB?style=for-the-badge)
![Control](https://img.shields.io/badge/Control-Sensorless%206--Step-8250DF?style=for-the-badge)
![PWM](https://img.shields.io/badge/PWM-16%20kHz-2EA44F?style=for-the-badge)
![PCB](https://img.shields.io/badge/PCB-2--Layer-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Prototype-yellow?style=for-the-badge)

</p>

---

## Table of Contents

- [Overview](#overview)
- [Key Specifications](#key-specifications)
- [System Architecture](#system-architecture)
- [Operating Principle](#operating-principle)
- [Design Requirements](#design-requirements)
- [Component Selection](#component-selection)
- [Hardware Architecture](#hardware-architecture)
- [Power Input Stage](#power-input-stage)
- [Microcontroller](#microcontroller)
- [Gate Drivers](#gate-drivers)
- [Power MOSFETs](#power-mosfets)
- [BEMF Sensing](#bemf-sensing)
- [Virtual Neutral](#virtual-neutral)
- [Hardware Pinout](#hardware-pinout)
- [Mathematical Calculations](#mathematical-calculations)
  - [BEMF Voltage Divider](#bemf-voltage-divider)
  - [Maximum BEMF Sense Voltage](#maximum-bemf-sense-voltage)
  - [Virtual Neutral Calculation](#virtual-neutral-calculation)
  - [Bootstrap Capacitor](#bootstrap-capacitor)
  - [Crystal Load Capacitors](#crystal-load-capacitors)
  - [PWM Frequency](#pwm-frequency)
  - [PCB Trace Current Capacity](#pcb-trace-current-capacity)
- [Six-Step Commutation](#six-step-commutation)
- [Commutation Table](#commutation-table)
- [Firmware Architecture](#firmware-architecture)
- [RC PWM Control](#rc-pwm-control)
- [Manual Button Control](#manual-button-control)
- [PCB Engineering](#pcb-engineering)
  - [Power Routing](#power-routing)
  - [Copper Reinforcement](#copper-reinforcement)
  - [Via Stitching](#via-stitching)
  - [Spatial Zoning](#spatial-zoning)
  - [Grounding Strategy](#grounding-strategy)
- [Thermal Design](#thermal-design)
- [EMI and Noise Considerations](#emi-and-noise-considerations)
- [Hardware Verification](#hardware-verification)
- [Initial Power-Up](#initial-power-up)
- [Gate Drive Verification](#gate-drive-verification)
- [Motor Connection and Testing](#motor-connection-and-testing)
- [Fault Protection](#fault-protection)
- [Project Structure](#project-structure)
- [Engineering Summary](#engineering-summary)
- [Design Philosophy](#design-philosophy)
- [Development Status](#development-status)
- [Safety](#safety)
- [License](#license)
- [Final Notes](#final-notes)

---

# Overview

**TRIDENT 328P** is an open-source, sensorless **3-phase Brushless DC (BLDC) Electronic Speed Controller (ESC)** based on the **ATmega328P-AU** microcontroller.

The controller is designed around a conventional six-step trapezoidal BLDC commutation strategy with sensorless rotor-position detection using the motor's **Back Electromotive Force (BEMF)**.

The design integrates:

- ATmega328P-AU microcontroller
- Sensorless six-step commutation
- BEMF zero-crossing detection
- Internal analog comparator
- Three IR2104S gate drivers
- Six IRLZ44N power MOSFETs
- 16 kHz PWM
- RC PWM input
- Manual speed control
- Three-phase BEMF sensing
- Virtual neutral generation
- Hardware shutdown inputs
- High-current PCB power routing
- Dedicated analog and digital sections

---

# Key Specifications

| Parameter | Specification |
|:---|---:|
| **Project** | TRIDENT 328P |
| **Controller** | Sensorless 3-Phase BLDC ESC |
| **Microcontroller** | ATmega328P-AU |
| **Input Voltage** | 8–16 V |
| **Recommended Battery** | 3S–4S LiPo |
| **Continuous Current Target** | 15 A |
| **Peak Current Target** | 25–30 A |
| **Peak Duration** | < 10 s |
| **Logic Supply** | 5 V |
| **PWM Frequency** | 16 kHz |
| **Commutation** | Sensorless 6-Step |
| **Rotor Position Feedback** | BEMF |
| **BEMF Detection** | Analog Comparator |
| **Gate Drivers** | 3 × IR2104S |
| **Power MOSFETs** | 6 × IRLZ44N |
| **RC PWM Input** | 1000–2000 µs |
| **Crystal Frequency** | 16 MHz |
| **PCB Layers** | 2 |
| **Copper Thickness** | 35 µm / 1 oz |
| **PCB Size** | 120 × 100 mm |
| **BEMF Divider** | 39 kΩ / 10 kΩ |
| **Bootstrap Capacitor** | 2.2 µF |
| **Control Methods** | RC PWM / Buttons |

---

# System Architecture

The complete TRIDENT 328P system can be divided into five major sections:

1. Power input
2. Three-phase power stage
3. Gate-drive stage
4. Microcontroller and control electronics
5. Analog BEMF sensing

```text
                          TRIDENT 328P
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
           ▼                  ▼                  ▼
      POWER STAGE        CONTROL STAGE      ANALOG STAGE
           │                  │                  │
           │                  │                  │
     Battery Input        ATmega328P-AU      BEMF Sensing
     Bulk Capacitors      Hardware PWM       Virtual Neutral
     MOSFET Bridge        Interrupts         Comparator
           │                  │                  │
           ▼                  ▼                  ▼
        PHASE A           Gate Drive         Zero Crossing
        PHASE B            Control             Detection
        PHASE C                │
           │                   ▼
           ▼              IR2104S × 3
      BLDC MOTOR               │
                               ▼
                           MOSFET × 6
