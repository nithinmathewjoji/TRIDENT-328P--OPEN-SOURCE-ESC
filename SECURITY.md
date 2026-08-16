# Security & Hardware Safety Policy

## Supported Hardware & Firmware Versions

As an open-source hardware Electronic Speed Controller (ESC) project, security and safety updates cover firmware releases, schematic revisions, and critical hardware errata/fixes.

| Hardware / Firmware Revision | Supported          | Release Status | Notes |
| :--- | :--- | :--- | :--- |
| **TRIDENT-328P Rev 1.0 (Current)** | :white_check_mark: | Active Hardware Release | Active patches for firmware, commutation timing, and PCB errata |
| **TRIDENT-328P Pre-release / Proto** | :x: | Deprecated | Superseded by Rev 1.0 production Gerber release |
| **TRIDENT V2 (STM32 / FOC)** | :construction: | In Planning / Development | Future branch |

---

## Scope & Critical Safety Vectors

Given that **TRIDENT-328P** controls high-current switching ($15\text{A} - 30\text{A}$) and inductive loads at up to $24\text{V}$, vulnerabilities are evaluated based on **physical safety, thermal runaway risk, and firmware stability**:

* **Hardware Shoot-Through / Driver Dead-Time Faults:** Logic flaws or race conditions in gate drive switching that could cause simultaneous high-side and low-side conduction.
* **Commutation Lock-up & Stall Protection:** Failure of zero-crossing BEMF detection to detect rotor stalls, leading to excessive continuous DC injection and thermal runaway.
* **PWM Overflow / Out-of-Bounds Throttle Input:** Signal decoding errors on the `RC_PWM_IN` (`INT1`) or button interfaces causing unintended motor acceleration or uncommanded full throttle.
* **Electrical & Thermal Boundaries:** Schematic or component margin errors that could cause component failure within the documented operating envelope ($8\text{V} - 24\text{V}$).

---

## Reporting a Vulnerability or Critical Safety Issue

If you discover a firmware security vulnerability, an electrical hazard, or a hardware design flaw that poses safety or operational risks, please follow these reporting procedures:

### 1. Where to Report
* **Private Disclosure (Preferred for critical safety/hardware risks):**  
  Open a **Private Security Advisory** on the project's GitHub repository under `Security` > `Advisories` > `Report a vulnerability`, or email the maintainer directly.
* **Non-Hazardous Bugs / Minor Documentation Issues:**  
  General firmware bugs, silkscreen typos, or minor non-destructive issues can be filed directly via public **GitHub Issues**.

### 2. Information to Include in Your Report
To help quickly diagnose and reproduce the issue, please include:
* **Hardware Details:** Board revision, operating DC supply voltage ($V_{CC}$), motor specifications (KV rating, pole count, phase resistance/inductance).
* **Firmware Configuration:** Control mode (RC Receiver PWM vs. Onboard Buttons), PWM frequency, and commutation settings.
* **Observed vs. Expected Behavior:** Oscilloscope/logic analyzer captures (especially of Phase voltages, gate signals, or comparator pins) if available.
* **Steps to Reproduce:** Exact sequence of throttle inputs or electrical test conditions that triggered the issue.

### 3. Response & Resolution Process

To ensure rapid triage of potential electrical, thermal, and firmware hazards, all reported vulnerabilities follow a structured 5-stage resolution lifecycle:
### Stage 1: Ingestion & Initial Acknowledgement (Within 48 Hours)
* The maintainer logs the report privately and verifies the submitted technical details (operating voltage, motor KV/inductance, throttle input mode, oscilloscope/logic analyzer traces).
* A response is sent to the reporter confirming receipt, assigning a tracking identifier (e.g., `SEC-TRIDENT-2026-01`), and establishing a private communication channel.

### Stage 2: Severity Classification & Impact Triage

Reports are categorized based on risk to physical safety, hardware destruction, or firmware control failure:

| Severity Level | Definition & Criteria | Target Fix Window | Example Scenarios |
| :--- | :--- | :--- | :--- |
| **Critical** | Physical safety risk, active fire/thermal hazard, uncommanded full-throttle acceleration. | **< 72 Hours** | Dead-time elimination (shoot-through), RC PWM signal decode overflow causing runaway motor rotation. |
| **High** | Component destruction risk, stalled rotor thermal runaway, commutation lock-up under load. | **< 7 Days** | BEMF zero-crossing comparator lock causing continuous DC injection into stationary stator windings. |
| **Medium** | Erroneous telemetry, unexpected speed dropouts, mode-switching glitches without hardware damage. | **< 14 Days** | Switch debounce race conditions on `SW4` (`C_SELECT`), button stepping jitter on `PC3`/`PC4`. |
| **Low / Informational** | Documentation inaccuracies, silkscreen typos, non-critical parameter misalignments. | **Next Minor Release** | Silkscreen label errors, non-optimal default timer prescaler recommendations. |

### Stage 3: Hardware & Firmware Reproduction
* **Simulation & Bench Testing:** The maintainer reproduces the reported fault condition using a current-limited bench supply, logic analyzer, and oscilloscope connected to the Gate, Phase, and `AIN0`/`AIN1` test points.
* **Root Cause Determination:** The issue is isolated to one of three domains:
  1. **Firmware Register Logic:** (e.g., Timer OCR register update during active PWM cycle, comparator interrupt latency).
  2. **PCB Layout / Electrical Domain:** (e.g., Parasitic trace inductance causing gate ringing, ground bounce on analog virtual neutral).
  3. **Operational Exceedance:** (e.g., Motor stator saturation, exceeding maximum $V_{CC}$ of 24V).

### Stage 4: Remediation, Patching & Validation
* **Firmware Fixes:** Developed in a private branch, verified for zero-regression on both RC Receiver and manual button operating modes.
* **Hardware Errata & PCB Updates:** If the issue requires a physical hardware patch:
  * A **Hardware Advisory / Errata Note** is drafted describing rework instructions (e.g., adding an RC snubber across phase nodes, changing gate resistor values).
  * The KiCad schematic and PCB layout files are updated for the next sub-revision.

### Stage 5: Coordinated Disclosure & Release
* **Release:** A new GitHub Release is tagged with updated source code, compiled `.hex` binaries, revised Gerber archives, and a detailed changelog.
* **Advisory Publication:** A public GitHub Security Advisory is published outlining:
  * Description of the vulnerability and affected revisions.
  * Risk evaluation and mitigation steps.
  * Direct links to patched firmware or hardware errata workarounds.
* **Reporter Credit:** The original reporter is formally credited in the release notes (unless anonymity is requested).

---

## Disclaimer & Safe Operating Practices

* **Always use a current-limited DC bench power supply** ($< 1\text{A}$ limit) during initial bring-up, flashing, and testing before connecting high-discharge LiPo batteries.
* **Remove motor propellers or disconnect mechanical loads** when flashing new firmware or testing open-loop commutation routines.
* The hardware has **no reverse polarity protection**—ensure correct power terminal orientation prior to energizing the board.
