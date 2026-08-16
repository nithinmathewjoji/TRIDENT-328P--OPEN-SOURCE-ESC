# Contributing to TRIDENT-328P

Thank you for your interest in contributing to **TRIDENT-328P**! As an open-source hardware (OSHW) and embedded firmware project, contributions of all kinds—hardware optimization, firmware patches, documentation improvements, and safety errata—are welcomed.

To ensure hardware safety, technical accuracy, and reproducible designs, please follow these guidelines when proposing changes.

---

## Table of Contents
1. [Code of Conduct](#1-code-of-conduct)
2. [How to Contribute](#2-how-to-contribute)
   * [Reporting Bugs & Electrical Errata](#reporting-bugs--electrical-errata)
   * [Suggesting Features & Enhancements](#suggesting-features--enhancements)
   * [Submitting Pull Requests](#submitting-pull-requests)
3. [Hardware Contribution Standards (EDA / PCB)](#3-hardware-contribution-standards-eda--pcb)
   * [KiCad Guidelines](#kicad-guidelines)
   * [Layout & DRC Rules](#layout--drc-rules)
   * [BOM & Component Sourcing](#bom--component-sourcing)
4. [Firmware Contribution Standards](#4-firmware-contribution-standards)
   * [Coding Standards & Register-Level Hygiene](#coding-standards--register-level-hygiene)
   * [Interrupt & Timing Constraints](#interrupt--timing-constraints)
5. [Bench Testing & Hardware Verification](#5-bench-testing--hardware-verification)
6. [Git Workflow & Commit Conventions](#6-git-workflow--commit-conventions)

---

## 1. Code of Conduct

We are committed to providing a welcoming, constructive, and harassment-free community. Please be respectful, collaborative, and considerate when reviewing pull requests or discussing technical implementations.

---

## 2. How to Contribute

### Reporting Bugs & Electrical Errata
* For safety hazards, shoot-through risks, or electrical boundary flaws, review our [`SECURITY.md`](SECURITY.md) before opening a public issue.
* For general firmware bugs or documentation typos, open a **GitHub Issue** using the provided templates.
* Include detailed reproduction steps: DC supply voltage ($V_{CC}$), motor specifications (KV, pole count, inductance), throttle mode, and oscilloscope/logic analyzer traces if available.

### Suggesting Features & Enhancements
* Open a feature request issue to discuss major hardware revisions (e.g., current sensing shunts, driver alternatives, FOC roadmap) before spending hours routing a PCB.

### Submitting Pull Requests
1. Fork the repository and create a descriptive feature branch:
   ```bash
   git checkout -b feature/optimize-bemf-filter

   ## 3. Hardware Contribution Standards (EDA / PCB)

### KiCad Guidelines
* **EDA Suite:** The official hardware source is maintained in **KiCad 8.x / latest stable**. Avoid using vendor-locked or proprietary CAD tools.
* **Component Libraries:** Use local project symbols and footprints or standard KiCad library components. Embed custom footprints within the repository's `lib/` directory so designs remain 100% portable.
* **Schematic Hierarchy & Net Labels:** Keep schematics partitioned into clear functional blocks (Power Entry, MCU Core, Gate Drivers, Power Bridge, BEMF Divider). Use global or hierarchical labels with explicit net names (`P_SENSE_1`, `SD_1`, `V_NEUTRAL`).

### Layout & DRC Rules
* **Design Rule Check (DRC):** All submitted `.kicad_pcb` files must pass DRC with **0 Errors** and **0 Unconnected Nets**.
* **High-Current Traces:**
  * Power rails ($V_{CC}$, GND, Phase outputs) must follow defined high-current clearance and trace-width standards ($\ge 3.0\text{ mm}$ or polygon zones).
  * Maintain solder mask openings (`F.Mask` / `B.Mask`) over power lines for manual solder reinforcement where intended.
* **Ground Pours & Stitching:** Avoid floating copper islands. Maintain a continuous ground plane and ensure via stitching around switching loops and thermal dissipators.
* **Silkscreen Legibility:**
  * Silkscreen text must not overlap exposed solder pads or via holes.
  * Use a minimum text size of $1.0\text{ mm}$ height and $0.15\text{ mm}$ line thickness for production readability.

### BOM & Component Sourcing
* Prefer easily sourceable, active components with broad multi-distributor availability (LCSC, Mouser, DigiKey).
* Update the project `BOM.csv` with manufacturer part numbers (MPN), footprints, and tolerance specs whenever components are altered.

---

## 4. Firmware Contribution Standards

### Coding Standards & Register-Level Hygiene
* **Target Architecture:** ATmega328P (8-bit AVR @ 16 MHz).
* **Efficiency:** Firmware is written in clean, modern C/C++. Favor direct AVR register manipulation (`TCCR1A`, `ACSR`, `ADMUX`) where timing precision is critical, but maintain readable macro abstractions and clean variable naming.
* **No Blocking Delays in ISRs:** Never use `_delay_ms()` or blocking loops inside interrupt service routines (`ISR`).
* **RAM Optimization:** The ATmega328P has only 2 KB of SRAM. Avoid dynamic memory allocation (`malloc`/`new`), heavy C++ standard libraries, or large heap buffers.

### Interrupt & Timing Constraints
* **Commutation Timing:** Back-EMF zero-crossing comparator interrupts (`ANALOG_COMP_vect`) and Timer capture routines must execute in predictable, bounded clock cycles to prevent commutation jitter.
* **Dead-Time & Shoot-Through Safety:** Never modify phase drive output registers without verifying that hardware/software dead-time is preserved.

---

## 5. Bench Testing & Hardware Verification

Before submitting firmware or hardware PRs that affect motor drive behavior:

1. **Current-Limited Bench Supply Test:** Verify functionality on a DC bench power supply with a current limit set to $\le 500\text{ mA}$ with no motor load.
2. **Signal Verification:** Confirm gate drive signals and dead-time on an oscilloscope or logic analyzer.
3. **No-Load Spin-Up:** Test open-loop ramp-up and closed-loop BEMF handover on an unloaded BLDC motor.
4. **Thermal Stability:** Monitor MOSFET and regulator temperatures under light load.

> *Include test results, waveform screenshots, or test setup details in your PR description.*

---

## 6. Git Workflow & Commit Conventions

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

* `feat(hw): add reverse polarity protection circuit`
* `fix(fw): eliminate phase advance jitter at high RPM`
* `docs: update BEMF voltage divider calculations in README`
* `refactor(driver): streamline Timer1 commutation lookup table`
* `chore: update gerber generation scripts`

---

## Questions or Need Help?

If you have questions about architecture or need guidance on setting up the KiCad project, feel free to open a [GitHub Discussion](https://github.com/) or reach out via our community issue tracker!
