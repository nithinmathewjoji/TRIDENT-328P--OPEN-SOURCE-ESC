# TRIDENT 328P

<div align="center">

<pre>
  ████████╗██████╗ ██╗██████╗ ███████╗███╗   ██╗████████╗
  ╚══██╔══╝██╔══██╗██║██╔══██╗██╔════╝████╗  ██║╚══██╔══╝
     ██║   ██████╔╝██║██║  ██║█████╗  ██╔██╗ ██║   ██║
     ██║   ██╔══██╗██║██║  ██║██╔══╝  ██║╚██╗██║   ██║
     ██║   ██║  ██║██║██████╔╝███████╗██║ ╚████║   ██║
     ╚═╝   ╚═╝  ╚═╝╚═╝╚═════╝ ╚══════╝╚═╝  ╚═══╝   ╚═╝
</pre>

<h2>Sensorless 3-Phase BLDC Electronic Speed Controller</h2>

<p>
  <strong>Open Hardware • ATmega328P • Sensorless 6-Step Commutation</strong>
</p>

<p>

![MCU](https://img.shields.io/badge/MCU-ATmega328P--AU-0A7EA4?style=for-the-badge)
![ESC](https://img.shields.io/badge/ESC-3--Phase%20BLDC-1F6FEB?style=for-the-badge)
![Control](https://img.shields.io/badge/Control-Sensorless%206--Step-8250DF?style=for-the-badge)
![PWM](https://img.shields.io/badge/PWM-16%20kHz-2EA44F?style=for-the-badge)
![PCB](https://img.shields.io/badge/PCB-2--Layer-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Prototype-yellow?style=for-the-badge)

</p>

</div>

---

## Table of Contents

* [Overview](#overview)
* [Key Specifications](#key-specifications)
* [System Architecture](#system-architecture)
* [Operating Principle](#operating-principle)
* [Design Requirements](#design-requirements)
* [Component Selection](#component-selection)
* [Hardware Architecture](#hardware-architecture)
* [Power Input Stage](#power-input-stage)
* [Microcontroller](#microcontroller)
* [Gate Drivers](#gate-drivers)
* [Power MOSFETs](#power-mosfets)
* [BEMF Sensing](#bemf-sensing)
* [Virtual Neutral](#virtual-neutral)
* [Hardware Pinout](#hardware-pinout)
* [Mathematical Calculations](#mathematical-calculations)

  * [BEMF Voltage Divider](#bemf-voltage-divider)
  * [Maximum BEMF Sense Voltage](#maximum-bemf-sense-voltage)
  * [Virtual Neutral Calculation](#virtual-neutral-calculation)
  * [Bootstrap Capacitor](#bootstrap-capacitor)
  * [Crystal Load Capacitors](#crystal-load-capacitors)
  * [PWM Frequency](#pwm-frequency)
  * [PCB Trace Current Capacity](#pcb-trace-current-capacity)
* [Six-Step Commutation](#six-step-commutation)
* [Commutation Table](#commutation-table)
* [Firmware Architecture](#firmware-architecture)
* [RC PWM Control](#rc-pwm-control)
* [Manual Button Control](#manual-button-control)
* [PCB Engineering](#pcb-engineering)

  * [Power Routing](#power-routing)
  * [Copper Reinforcement](#copper-reinforcement)
  * [Via Stitching](#via-stitching)
  * [Spatial Zoning](#spatial-zoning)
  * [Grounding Strategy](#grounding-strategy)
* [Thermal Design](#thermal-design)
* [EMI and Noise Considerations](#emi-and-noise-considerations)
* [Hardware Verification](#hardware-verification)
* [Initial Power-Up](#initial-power-up)
* [Gate Drive Verification](#gate-drive-verification)
* [Motor Connection and Testing](#motor-connection-and-testing)
* [Fault Protection](#fault-protection)
* [Project Structure](#project-structure)
* [Engineering Summary](#engineering-summary)
* [Design Philosophy](#design-philosophy)
* [Development Status](#development-status)
* [Safety](#safety)
* [License](#license)
* [Final Notes](#final-notes)

---

# Overview

**TRIDENT 328P** is an open-source, sensorless **3-phase Brushless DC (BLDC) Electronic Speed Controller (ESC)** based on the **ATmega328P-AU** microcontroller.

The controller is designed around a conventional six-step trapezoidal BLDC commutation strategy with sensorless rotor-position detection using the motor's **Back Electromotive Force (BEMF)**.

The design integrates:

* ATmega328P-AU microcontroller
* Sensorless six-step commutation
* BEMF zero-crossing detection
* Internal analog comparator
* Three IR2104S gate drivers
* Six IRLZ44N power MOSFETs
* 16 kHz PWM
* RC PWM input
* Manual speed control
* Three-phase BEMF sensing
* Virtual neutral generation
* Hardware shutdown inputs
* High-current PCB power routing
* Dedicated analog and digital sections

---

# Key Specifications

| Parameter                     |               Specification |
| :---------------------------- | --------------------------: |
| **Project**                   |                TRIDENT 328P |
| **Controller**                | Sensorless 3-Phase BLDC ESC |
| **Microcontroller**           |               ATmega328P-AU |
| **Input Voltage**             |                      8–16 V |
| **Recommended Battery**       |                  3S–4S LiPo |
| **Continuous Current Target** |                        15 A |
| **Peak Current Target**       |                     25–30 A |
| **Peak Duration**             |                      < 10 s |
| **Logic Supply**              |                         5 V |
| **PWM Frequency**             |                      16 kHz |
| **Commutation**               |           Sensorless 6-Step |
| **Rotor Position Feedback**   |                        BEMF |
| **BEMF Detection**            |           Analog Comparator |
| **Gate Drivers**              |                 3 × IR2104S |
| **Power MOSFETs**             |                 6 × IRLZ44N |
| **RC PWM Input**              |                1000–2000 µs |
| **Crystal Frequency**         |                      16 MHz |
| **PCB Layers**                |                           2 |
| **Copper Thickness**          |                35 µm / 1 oz |
| **PCB Size**                  |                120 × 100 mm |
| **BEMF Divider**              |               39 kΩ / 10 kΩ |
| **Bootstrap Capacitor**       |                      2.2 µF |
| **Control Methods**           |            RC PWM / Buttons |

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
```

---

# Operating Principle

TRIDENT 328P uses sensorless six-step commutation.

Unlike a sensored BLDC controller, no Hall-effect sensors are required. Instead, the controller observes the BEMF generated by the motor's floating phase.

During each commutation state:

1. One phase is driven high.
2. One phase is driven low.
3. One phase is left floating.
4. The floating phase generates a BEMF voltage.
5. The BEMF voltage is compared against the virtual neutral.
6. The zero crossing is detected.
7. The firmware waits for the required commutation delay.
8. The next commutation state is activated.

```text
        BLDC MOTOR
      ┌─────────────┐
      │             │
 PHASE A ───────────┤
 PHASE B ───────────┤
 PHASE C ───────────┤
      │             │
      └─────────────┘
             │
             ▼
      Floating Phase
             │
             ▼
       BEMF Divider
             │
             ▼
      Analog Comparator
             │
             ▼
        ATmega328P-AU
             │
             ▼
      Commutation Timing
```

---

# Design Requirements

| Requirement            |            Target |
| ---------------------- | ----------------: |
| **Input Voltage**      |            8–16 V |
| **Continuous Current** |              15 A |
| **Peak Current**       |           25–30 A |
| **Peak Duration**      |            < 10 s |
| **Logic Voltage**      |               5 V |
| **PWM Frequency**      |            16 kHz |
| **Commutation**        | Sensorless 6-Step |
| **Position Detection** |              BEMF |
| **RC Input**           |      1000–2000 µs |
| **PCB Size**           |      120 × 100 mm |
| **PCB Layers**         |                 2 |
| **Copper Thickness**   |             35 µm |
| **MCU Clock**          |            16 MHz |

---

# Component Selection

| Function                  | Component          | Purpose                         |
| ------------------------- | ------------------ | ------------------------------- |
| **Microcontroller**       | ATmega328P-AU      | Main controller                 |
| **Gate Driver**           | IR2104S × 3        | High-side / low-side gate drive |
| **Power MOSFET**          | IRLZ44N × 6        | Three-phase bridge              |
| **Voltage Regulator**     | LM7805CT           | 5 V logic supply                |
| **Bootstrap Diode**       | SS14               | Bootstrap charging              |
| **Crystal**               | 16 MHz             | MCU clock                       |
| **Bulk Capacitors**       | 2200 µF / 50 V × 2 | DC bus energy storage           |
| **BEMF Divider**          | 39 kΩ / 10 kΩ      | Phase voltage attenuation       |
| **Gate Resistors**        | Design dependent   | Gate current control            |
| **Decoupling Capacitors** | 100 nF             | High-frequency supply bypass    |

---

# Hardware Architecture

```text
┌─────────────────────────────────────────────────────────────────────┐
│                         TRIDENT 328P                                │
├──────────────────────┬──────────────────────┬───────────────────────┤
│ POWER INPUT          │ GATE DRIVE           │ POWER BRIDGE          │
│                      │                      │                       │
│ Battery + / -        │ IR2104S × 3         │ IRLZ44N × 6           │
│ Bulk Capacitors      │ Bootstrap Network    │ Phase A               │
│ Voltage Regulation   │ Gate Resistors       │ Phase B               │
│                      │ Shutdown Inputs      │ Phase C               │
├──────────────────────┼──────────────────────┼───────────────────────┤
│ DIGITAL CONTROL      │ ANALOG SENSING       │ USER INTERFACE        │
│                      │                      │                       │
│ ATmega328P-AU        │ BEMF Dividers        │ RC PWM                │
│ Hardware PWM         │ Virtual Neutral      │ Speed Up              │
│ Timers               │ Analog Comparator    │ Speed Down            │
│ Interrupts           │ ADC Measurements     │ Mode Select           │
└──────────────────────┴──────────────────────┴───────────────────────┘
```

---

# Power Input Stage

The DC input stage supplies both the three-phase bridge and the low-voltage electronics.

```text
              BATTERY
             8–16 V DC
                  │
                  ▼
        ┌─────────────────┐
        │ INPUT PROTECTION│
        └────────┬────────┘
                 │
                 ▼
        ┌─────────────────┐
        │ BULK CAPACITORS │
        │ 2200 µF × 2     │
        └────────┬────────┘
                 │
           ┌─────┴─────┐
           │           │
           ▼           ▼
       POWER STAGE  5 V REGULATOR
           │           │
           ▼           ▼
      MOSFET BRIDGE  ATmega328P
```

Bulk capacitance should be located physically close to the MOSFET bridge.

The objective is to minimize:

* Supply-loop inductance
* Voltage overshoot
* Switching noise
* DC bus ripple

---

# Microcontroller

## ATmega328P-AU

The ATmega328P-AU is the central control device of TRIDENT 328P.

The MCU performs:

* PWM generation
* Commutation sequencing
* BEMF detection
* ADC measurements
* RC PWM measurement
* Button processing
* Fault handling
* Status indication
* Timing control

## Analog Comparator

The internal analog comparator is used for BEMF zero-crossing detection.

The floating phase voltage $V_{\text{BEMF}}$ is compared directly against the virtual neutral voltage $V_{\text{NEUTRAL}}$.

When the floating phase crosses the virtual neutral voltage, the comparator changes state.

This transition is used as a rotor-position timing reference.

---

# Gate Drivers

## IR2104S

Three IR2104S devices are used. Each driver controls one motor phase.

```text
             DC BUS
                │
          ┌─────┴─────┐
          │           │
       HIGH SIDE   BOOTSTRAP
       MOSFET         │
          │           │
          └─────┬─────┘
                │
              PHASE
                │
          ┌─────┴─────┐
          │           │
       LOW SIDE    GATE DRIVER
       MOSFET         │
          │           │
          └─────┬─────┘
                │
               GND
```

Each driver provides:

* High-side gate drive
* Low-side gate drive
* Bootstrap operation
* Shutdown control
* Gate-drive timing
* Dead-time control

---

# Power MOSFETs

## IRLZ44N

Six IRLZ44N N-channel MOSFETs form the three-phase inverter.

```text
                         DC+
                          │
               ┌──────────┼──────────┐
               │          │          │
              QH1        QH2        QH3
               │          │          │
              PH-A       PH-B       PH-C
               │          │          │
              QL1        QL2        QL3
               │          │          │
               └──────────┼──────────┘
                          │
                         GND
```

The MOSFETs must be evaluated for:

* Drain-source voltage
* Continuous current
* Pulse current
* Gate charge
* $R_{\text{DS(on)}}$
* Switching losses
* Thermal performance

---

# BEMF Sensing

Each motor phase is monitored through a resistor divider.

The divider reduces the motor phase voltage to a level suitable for the ATmega328P sensing circuitry.

The selected divider components are:

$$
R_{\text{TOP}} = 39,\text{k}\Omega
$$

$$
R_{\text{BOTTOM}} = 10,\text{k}\Omega
$$

The divider is applied independently to each phase:

```text
              MOTOR PHASE
                   │
                   │
                 R_TOP
                39 kΩ
                   │
                   ├──────────► V_SENSE
                   │
              R_BOTTOM
                10 kΩ
                   │
                  GND
```

---

# Virtual Neutral

The three sensed phase voltages are combined to form a virtual neutral.

For equal weighting:

$$
V_{\text{NEUTRAL}}
==================

\frac{
V_{\text{SENSE1}}
+
V_{\text{SENSE2}}
+
V_{\text{SENSE3}}
}{3}
$$

The virtual neutral provides the reference for BEMF zero-crossing detection.

---

# Hardware Pinout

## ATmega328P-AU — TQFP-32

| Pin | Port | Net Name     | Function          |
| --: | ---- | ------------ | ----------------- |
|   1 | PD3  | RC_PWM_IN    | RC PWM / INT1     |
|   2 | PD4  | SD_2         | Phase 2 shutdown  |
|   3 | GND  | GND          | Ground            |
|   4 | VCC  | +5V          | Logic supply      |
|   5 | GND  | GND          | Ground            |
|   6 | VCC  | +5V          | Logic supply      |
|   7 | PB6  | XTAL1        | Crystal           |
|   8 | PB7  | XTAL2        | Crystal           |
|   9 | PD5  | SD_3         | Phase 3 shutdown  |
|  10 | PD6  | V_NEUTRAL    | Comparator input  |
|  11 | PD7  | NC           | Spare GPIO        |
|  12 | PB0  | NC           | Spare GPIO        |
|  13 | PB1  | IN_1         | Phase 1 PWM       |
|  14 | PB2  | IN_2         | Phase 2 PWM       |
|  15 | PB3  | IN_3         | Phase 3 PWM       |
|  16 | PB4  | NC           | Spare GPIO        |
|  17 | PB5  | STATUS_LED   | Status LED        |
|  18 | AVCC | +5V          | Analog supply     |
|  19 | ADC6 | NC           | Spare ADC         |
|  20 | AREF | AREF         | Analog reference  |
|  21 | GND  | GND          | Ground            |
|  22 | ADC7 | NC           | Spare ADC         |
|  23 | PC0  | P-SENSE-1    | Phase 1 ADC       |
|  24 | PC1  | P-SENSE-2    | Phase 2 ADC       |
|  25 | PC2  | P-SENSE-3    | Phase 3 ADC       |
|  26 | PC3  | BTN_SPEED_UP | Speed-up button   |
|  27 | PC4  | BTN_SPEED_DN | Speed-down button |
|  28 | PC5  | C_SELECT     | Control selection |
|  29 | PC6  | RESET        | MCU reset         |
|  30 | PD0  | FTDI_TX      | UART              |
|  31 | PD1  | FTDI_RX      | UART              |
|  32 | PD2  | SD_1         | Phase 1 shutdown  |

---

# Mathematical Calculations

## BEMF Voltage Divider

The BEMF sensing divider uses:

$$
R_{\text{TOP}} = 39,\text{k}\Omega
$$

$$
R_{\text{BOTTOM}} = 10,\text{k}\Omega
$$

The standard voltage-divider equation is:

$$
V_{\text{SENSE}}
================

V_{\text{PHASE}}
\left(
\frac{R_{\text{BOTTOM}}}
{R_{\text{TOP}} + R_{\text{BOTTOM}}}
\right)
$$

Substituting the selected resistor values:

$$
\begin{aligned}
V_{\text{SENSE}}
&=
V_{\text{PHASE}}
\left(
\frac{10,\text{k}\Omega}
{39,\text{k}\Omega + 10,\text{k}\Omega}
\right) \
&=
V_{\text{PHASE}}
\left(
\frac{10}{49}
\right) \
&\approx
0.20408 \cdot V_{\text{PHASE}}
\end{aligned}
$$

Therefore:

$$
\boxed{
V_{\text{SENSE}}
\approx
0.20408 \cdot V_{\text{PHASE}}
}
$$

---

## Maximum BEMF Sense Voltage

The maximum design input voltage is:

$$
V_{\text{PHASE,MAX}} = 16,\text{V}
$$

Using the calculated divider ratio:

$$
\begin{aligned}
V_{\text{SENSE,MAX}}
&=
16,\text{V}
\times
0.20408 \
&\approx
3.265,\text{V}
\end{aligned}
$$

Therefore:

$$
\boxed{
V_{\text{SENSE,MAX}}
\approx
3.27,\text{V}
}
$$

This provides a nominal sensing voltage well below the 5 V logic supply limit.

---

## Virtual Neutral Calculation

The virtual neutral is calculated from the three phase-sense voltages:

$$
\boxed{
V_{\text{NEUTRAL}}
==================

\frac{
V_{\text{SENSE1}}
+
V_{\text{SENSE2}}
+
V_{\text{SENSE3}}
}{3}
}
$$

The BEMF zero-crossing condition is defined as:

$$
\boxed{
V_{\text{BEMF}}
===============

V_{\text{NEUTRAL}}
}
$$

---

## Bootstrap Capacitor

The high-side MOSFET requires a gate voltage above its source voltage.

The IR2104S therefore uses a bootstrap capacitor.

Assuming:

* $Q_g \approx 63,\text{nC}$
* Additional charge allowance $Q_{\text{LS}} = 5,\text{nC}$
* Bootstrap supply current $I_{\text{IQBS}} = 50,\mu\text{A}$
* Maximum high-side on-time $t_{\text{HON,MAX}} = 200,\mu\text{s}$

The total estimated charge requirement is:

$$
\begin{aligned}
Q_{\text{TOTAL}}
&\approx
Q_g
+
Q_{\text{LS}}
+
(I_{\text{IQBS}} \cdot t_{\text{HON,MAX}}) \
&=
63,\text{nC}
+
5,\text{nC}
+
(50,\mu\text{A}
\times
200,\mu\text{s}) \
&\approx
78,\text{nC}
\end{aligned}
$$

Assuming a maximum bootstrap voltage drop:

$$
\Delta V_{\text{BOOT}} = 0.2,\text{V}
$$

The minimum bootstrap capacitance is:

$$
\begin{aligned}
C_{\text{BOOT,MIN}}
&\geq
\frac{Q_{\text{TOTAL}}}
{\Delta V_{\text{BOOT}}} \
&\geq
\frac{78,\text{nC}}
{0.2,\text{V}} \
&=
0.39,\mu\text{F}
\end{aligned}
$$

A practical selected value with additional operating margin is:

$$
\boxed{
C_{\text{BOOT,SELECTED}}
========================

2.2,\mu\text{F}
}
$$

---

## Crystal Load Capacitors

The MCU uses a:

$$
f_{\text{XTAL}} = 16,\text{MHz}
$$

Assuming:

$$
C_L = 20,\text{pF}
$$

and:

$$
C_{\text{STRAY}} = 4,\text{pF}
$$

The load-capacitance equation is:

$$
C_L
===

\frac{C_1 C_2}
{C_1 + C_2}
+
C_{\text{STRAY}}
$$

For equal load capacitors:

$$
C_1 = C_2 = C
$$

Therefore:

$$
\begin{aligned}
20,\text{pF}
&=
\frac{C}{2}
+
4,\text{pF} \
\frac{C}{2}
&=
16,\text{pF} \
C
&=
32,\text{pF}
\end{aligned}
$$

A practical standard selection is:

$$
\boxed{
C_1 = C_2 \approx 30\text{–}33,\text{pF}
}
$$

---

## PWM Frequency

The target PWM frequency is:

$$
f_{\text{PWM}} = 16,\text{kHz}
$$

Therefore:

$$
\begin{aligned}
T_{\text{PWM}}
&=
\frac{1}{16,000} \
&=
62.5,\mu\text{s}
\end{aligned}
$$

Therefore:

$$
\boxed{
T_{\text{PWM}} = 62.5,\mu\text{s}
}
$$

---

## PCB Trace Current Capacity

Using the standard IPC-2221 conductor relationship:

$$
I
=

k \cdot
(\Delta T)^{0.44}
\cdot
A^{0.725}
$$

For external conductors:

$$
k = 0.048
$$

Assuming:

$$
\Delta T = 30^\circ\text{C}
$$

$$
t_{\text{Cu}} = 35,\mu\text{m}
$$

which corresponds to 1 oz copper, and a target continuous current of:

$$
I = 15,\text{A}
$$

The calculated trace width is:

$$
\boxed{
W_{\text{TRACE}}
\approx
5.2,\text{mm}
}
$$

> **Note:** PCB current capacity is affected by copper thickness, temperature rise, trace geometry, ambient conditions, vias, solder reinforcement, connector resistance, and fabrication process. The calculated value should be treated as an engineering estimate and verified experimentally.

---

# Six-Step Commutation

Sensorless BLDC control uses six electrical commutation states.

Each state drives two motor phases and leaves the third phase floating.

```text
        PHASE A ───────────┐
                           │
        PHASE B ───────────┼──── BLDC MOTOR
                           │
        PHASE C ───────────┘

                 One phase floating
                           │
                           ▼
                       BEMF SENSE
                           │
                           ▼
                      ZERO CROSSING
                           │
                           ▼
                       COMMUTATION
```

---

# Commutation Table

| Step | High-Side Phase | Low-Side Phase | Floating Phase |
| ---: | --------------- | -------------- | -------------- |
|    1 | A               | B              | C              |
|    2 | A               | C              | B              |
|    3 | B               | C              | A              |
|    4 | B               | A              | C              |
|    5 | C               | A              | B              |
|    6 | C               | B              | A              |

---

# Firmware Architecture

The firmware is divided into several functional layers:

```text
┌─────────────────────────────────────┐
│           APPLICATION               │
│                                     │
│ RC PWM / Buttons / Speed Control    │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│          MOTOR CONTROL              │
│                                     │
│ Six-Step Commutation                │
│ Startup / Ramp / Running            │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│        POSITION DETECTION           │
│                                     │
│ BEMF Zero Crossing                  │
│ Comparator Interrupt                │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│           HARDWARE                  │
│                                     │
│ PWM / Timers / GPIO / ADC           │
└─────────────────────────────────────┘
```

## Firmware Startup Sequence

```text
                   POWER ON
                      │
                      ▼
             SYSTEM INITIALIZATION
                      │
                      ▼
                PWM INITIALIZATION
                      │
                      ▼
             ADC / COMPARATOR
                INITIALIZATION
                      │
                      ▼
                ROTOR ALIGNMENT
                      │
                      ▼
                 OPEN-LOOP RAMP
                      │
                      ▼
                 VALID BEMF ?
                  /       \
                NO         YES
                │           │
                ▼           ▼
             CONTINUE    CLOSED LOOP
             RAMPING     COMMUTATION
                            │
                            ▼
                       SPEED CONTROL
```

---

# RC PWM Control

TRIDENT 328P accepts a conventional RC servo-style PWM input with pulse width between:

$$
1000\text{–}2000,\mu\text{s}
$$

```text
1000 µs                              1500 µs                              2000 µs
   │                                    │                                    │
   ▼                                    ▼                                    ▼
MIN SPEED                           MID SPEED                           MAX SPEED
```

The input pulse width is measured using an external interrupt.

The controller maps the pulse width to a desired PWM duty cycle.

---

# Manual Button Control

Two buttons can be used for manual speed adjustment.

| MCU Pin | Function            |
| ------- | ------------------- |
| PC3     | Speed Up            |
| PC4     | Speed Down          |
| PC5     | Control Mode Select |

```text
             ┌───────────────┐
             │ CONTROL MODE  │
             └───────┬───────┘
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
         RC PWM MODE      BUTTON MODE
             │                │
             ▼                ▼
        Pulse Width       SPEED UP/DOWN
             │                │
             └───────┬────────┘
                     ▼
                SPEED COMMAND
```

---

# PCB Engineering

## Power Routing

The main high-current path follows:

```text
BATTERY + ──► BULK CAPACITORS ──► MOSFET BRIDGE ──► MOTOR PHASES
```

The return path provides a direct, low-impedance connection between the MOSFET bridge and battery negative.

---

## Copper Reinforcement

High-current PCB traces may use exposed copper routes for solder reinforcement:

```text
       SOLDER REINFORCEMENT
══════════════════════════════════════

       COPPER POWER TRACE
──────────────────────────────────────

               FR-4
══════════════════════════════════════
```

---

## Via Stitching

Ground stitching vias connect top and bottom ground regions:

```text
         TOP GROUND PLANE

══════════════════════════════════

        ●       ●       ●       ●
        │       │       │       │
        ●       ●       ●       ●
        │       │       │       │
        ●       ●       ●       ●

══════════════════════════════════

        BOTTOM GROUND PLANE
```

---

## Spatial Zoning

```text
┌─────────────────────────────────────────────────────────────────────┐
│                         TRIDENT 328P PCB                            │
├─────────────────────┬─────────────────────┬─────────────────────────┤
│ POWER INPUT         │ GATE DRIVE          │ POWER BRIDGE            │
│                     │                     │                         │
│ Battery             │ IR2104S × 3         │ MOSFET × 6              │
│ Bulk Capacitors     │ Bootstrap           │ Phase Outputs           │
│ Regulator           │ Gate Resistors      │ High Current Copper     │
├─────────────────────┼─────────────────────┼─────────────────────────┤
│ DIGITAL             │ ANALOG              │ MOTOR                   │
│                     │                     │                         │
│ ATmega328P          │ BEMF Dividers       │ Phase A                 │
│ Crystal             │ Virtual Neutral     │ Phase B                 │
│ Buttons             │ Comparator          │ Phase C                 │
│ RC Input            │ ADC                 │ Motor Connector         │
└─────────────────────┴─────────────────────┴─────────────────────────┘
```

---

## Grounding Strategy

```text
                         BATTERY -
                             │
                             ▼
                      ┌──────────────┐
                      │   STAR GND   │
                      └──────┬───────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
        ┌────────────────┐       ┌────────────────┐
        │  POWER GROUND  │       │ SIGNAL GROUND  │
        │                │       │                │
        │ MOSFET Bridge  │       │ ATmega328P     │
        │ Motor Return   │       │ BEMF Sensing   │
        │ Battery Return │       │ Comparator     │
        └────────────────┘       └────────────────┘
```

---

# Thermal Design

At 15 A continuous operation, heat dissipation is primarily governed by MOSFET conduction losses:

$$
P_{\text{COND}}
===============

I_{\text{RMS}}^2
\cdot
R_{\text{DS(on)}}
$$

Switching loss is approximated as:

$$
P_{\text{SW}}
\approx
\frac{1}{2}
\cdot
V_{\text{DS}}
\cdot
I_{\text{D}}
\cdot
(t_r + t_f)
\cdot
f_{\text{SW}}
$$

Total MOSFET loss per device:

$$
P_{\text{TOTAL}}
================

P_{\text{COND}}
+
P_{\text{SW}}
$$

---

# EMI and Noise Considerations

* Keep gate-drive and bootstrap loops as short as possible.
* Place decoupling capacitors adjacent to IC supply pins.
* Place DC-link bulk capacitors directly adjacent to the MOSFET bridge.
* Separate BEMF sensing traces from high $dV/dt$ switching nodes.
* Route analog ground lines back to the star ground point.
* Keep crystal traces short and away from power switching components.

---

# Hardware Verification

## Visual Inspection

* [ ] Check PCB for solder bridges.
* [ ] Check MOSFET orientation.
* [ ] Check IR2104S orientation.
* [ ] Check diode polarity.
* [ ] Check capacitor polarity.
* [ ] Check crystal orientation and placement.
* [ ] Check connector polarity.
* [ ] Inspect high-current copper paths.
* [ ] Inspect BEMF traces.
* [ ] Inspect ground connections.

---

## Resistance Checks

With power disconnected:

* [ ] Measure resistance between VCC and GND.
* [ ] Check for phase-to-ground shorts.
* [ ] Check for phase-to-phase shorts.
* [ ] Check MOSFET drain-source paths.
* [ ] Check regulator input/output for shorts.

---

## Low-Voltage Checks

Before connecting the motor:

* [ ] Verify 5 V regulator output.
* [ ] Verify MCU reset.
* [ ] Verify 16 MHz clock.
* [ ] Verify PWM output.
* [ ] Verify comparator operation.
* [ ] Verify BEMF divider outputs.
* [ ] Verify shutdown signals.
* [ ] Verify button inputs.
* [ ] Verify RC PWM input.

---

# Initial Power-Up

```text
                    +12 V
                      │
                      ▼
               ┌───────────────┐
               │ BENCH SUPPLY  │
               │               │
               │ CURRENT LIMIT │
               └───────┬───────┘
                       │
                       ▼
               ┌───────────────┐
               │ TRIDENT 328P  │
               └───────┬───────┘
                       │
                       ▼
                      +5 V
                       │
                    VERIFY
                       │
                       ▼
                MOTOR DISCONNECTED
```

Verify that the logic supply meets nominal specifications:

$$
\boxed{
V_{DD} \approx 5.0,\text{V}
}
$$

---

# Gate Drive Verification

Verify gate waveforms on an oscilloscope prior to motor attachment:

* High-side and low-side gate waveforms
* Dead-time interval
* Absence of shoot-through condition
* Gate amplitude levels

Target PWM parameters:

$$
f_{\text{PWM}} = 16,\text{kHz}
$$

$$
T_{\text{PWM}} = 62.5,\mu\text{s}
$$

---

# Motor Connection and Testing

```text
LOW VOLTAGE
     │
     ▼
LOW DUTY
     │
     ▼
VERIFY ROTATION
     │
     ▼
VERIFY BEMF
     │
     ▼
VERIFY COMMUTATION
     │
     ▼
INCREASE SPEED
     │
     ▼
INCREASE LOAD
     │
     ▼
THERMAL TEST
```

---

# Fault Protection

```text
FAULT DETECTED
      │
      ▼
PWM DISABLED
      │
      ▼
MOSFETS TURN OFF
      │
      ▼
MOTOR DISCONNECTED
      │
      ▼
ERROR STATE
```

---

# Project Structure

```text
TRIDENT-328P/
│
├── README.md
│
├── hardware/
│   ├── schematic/
│   │   ├── TRIDENT-328P.kicad_sch
│   │   └── TRIDENT-328P.pdf
│   │
│   ├── pcb/
│   │   ├── TRIDENT-328P.kicad_pcb
│   │   └── TRIDENT-328P-3D.step
│   │
│   ├── gerbers/
│   │   ├── copper/
│   │   ├── soldermask/
│   │   ├── silkscreen/
│   │   └── drill/
│   │
│   ├── bom/
│   │   └── TRIDENT-328P-BOM.csv
│   │
│   └── fabrication/
│       └── fabrication-notes.md
│
├── firmware/
│   ├── src/
│   ├── include/
│   ├── config/
│   └── README.md
│
├── docs/
│   ├── architecture/
│   ├── calculations/
│   ├── pcb-layout/
│   ├── testing/
│   └── measurements/
│
├── images/
│   ├── schematic.png
│   ├── pcb-top.png
│   ├── pcb-bottom.png
│   ├── 3d-render.png
│   └── block-diagram.png
│
└── LICENSE
```

---

# Engineering Summary

| Engineering Area                 | Implementation                  |
| -------------------------------- | ------------------------------- |
| **Project**                      | TRIDENT 328P                    |
| **MCU**                          | ATmega328P-AU                   |
| **Control Method**               | Sensorless 6-Step               |
| **Position Detection**           | BEMF Zero Crossing              |
| **BEMF Detection**               | Analog Comparator               |
| **Gate Drivers**                 | 3 × IR2104S                     |
| **Power MOSFETs**                | 6 × IRLZ44N                     |
| **PWM Frequency**                | 16 kHz                          |
| **Input Voltage**                | 8–16 V                          |
| **Continuous Current Target**    | 15 A                            |
| **Peak Current Target**          | 25–30 A                         |
| **BEMF Divider**                 | 39 kΩ / 10 kΩ                   |
| **Divider Ratio**                | $\approx 0.20408$               |
| **Maximum Sense Voltage @ 16 V** | $\approx 3.27,\text{V}$         |
| **Bootstrap Capacitor**          | 2.2 µF                          |
| **Crystal**                      | 16 MHz                          |
| **Crystal Capacitors**           | 30–33 pF                        |
| **Logic Supply**                 | 5 V                             |
| **PCB**                          | 2-Layer                         |
| **Copper**                       | 35 µm / 1 oz                    |
| **PCB Size**                     | 120 × 100 mm                    |
| **Ground Strategy**              | Star Ground                     |
| **PCB Zoning**                   | Power / Gate / Analog / Digital |
| **Control Input**                | RC PWM / Buttons                |

---

# Design Philosophy

TRIDENT 328P follows three primary engineering principles:

1. **Keep the power path short.**
2. **Keep the analog path quiet.**
3. **Let hardware peripherals handle timing-critical events.**

```text
                         TRIDENT 328P
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
            POWER           CONTROL         ANALOG
              │               │               │
         MOSFET Bridge    ATmega328P      BEMF Sense
         Bulk Capacitors  Hardware PWM    Virtual Neutral
         Motor Phases     Interrupts      Comparator
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                       SENSORLESS BLDC
                         COMMUTATION
```

---

# Development Status

| Area                      | Status       |
| ------------------------- | ------------ |
| Hardware Architecture     | ✅ Defined    |
| Component Selection       | ✅ Defined    |
| BEMF Network              | ✅ Defined    |
| Mathematical Calculations | ✅ Documented |
| PCB Engineering           | ✅ Documented |
| Firmware Architecture     | ✅ Defined    |
| Commutation Strategy      | ✅ Defined    |
| Verification Procedure    | ✅ Documented |
| Prototype Assembly        | 🧪 Testing   |
| Motor Validation          | 🧪 Testing   |
| Thermal Validation        | 🧪 Testing   |
| Production Validation     | ⏳ Pending    |

---

# Safety

> [!WARNING]
> **TRIDENT 328P controls a high-current three-phase power stage.**
>
> Incorrect assembly, incorrect gate-drive timing, inadequate thermal management, or an improperly connected battery can cause component failure, fire, or physical injury.

Always:

* Use current limiting during initial development.
* Verify PCB assembly before connecting a battery.
* Verify MOSFET and gate driver orientation.
* Verify dead-time and bootstrap operation.
* Use appropriate battery protection and fusing.
* Keep high-current paths short and properly sized.
* Do not rely solely on firmware for hardware safety.

---

# License

Hardware documentation and schematic files are released under the **CERN-OHL-S** license.

Firmware source code is released under the **MIT** license.

---

# Final Notes

This README documents the architecture, electrical calculations, PCB strategy, firmware design, and verification procedures for **TRIDENT 328P**.

All mathematical equations are written using LaTeX syntax compatible with GitHub Markdown rendering.
