---
name: hardware-hil-testing
description: Human-in-the-loop hardware testing protocol for firmware: coordinate AI↔human actions (button press, interrupts, LED patterns), timebox steps, collect structured observations/evidence, and output a TEST report mapped to AC/API/SEC. Use when host tests are insufficient and behavior must be confirmed on real hardware.
compatibility: opencode
metadata:
  owner: alex
  domain: firmware
  freedom: low
---

# Hardware Human-in-the-loop Testing

This skill defines a repeatable **human-in-the-loop (HiL)** protocol for on-board testing where the AI cannot directly observe the hardware and must rely on **structured human observations** (button press, interrupt trigger, LED blink patterns, serial/RTT output).

## When to use me
Use this skill when:
- A behavior must be verified on real hardware (timing, interrupts, DMA, peripherals).
- Tests require a human action gate (press/hold button, toggle jumper, power-cycle, observe LED).
- You need a deterministic, repeatable runbook and a standardized report for QA.

## What I produce
1) A step-by-step runbook with timeboxed checkpoints
2) A structured observation protocol for the human
3) A final TEST report snippet mapping results to `AC-xxx` and `TEST-xxx`

> Coordinate with:
> - skill({ name: "firmware-testing" }) for host tests + unsafe policy
> - skill({ name: "team-contract-ids" }) for AC/TEST reference discipline

---

# Core principle
**The human is the sensor.** The AI is the coordinator.
So we standardize:
- what the human should do
- what they should observe
- how they should report it
- what counts as pass/fail
- what to do on ambiguity

---

# Phase 0: Safety & preflight (mandatory)
Before any step runs, ask for:

## 0.1 Hardware context
Human must provide:
- Board / MCU model:
- Power source (USB/battery/bench):
- Wiring/peripherals connected:
- Anything that can overheat or short:

## 0.2 “Dangerous actions” warning
If a step requires:
- probing mains / high voltage
- hot-swapping power rails
- moving wires while powered
Then stop and request a safer procedure. Never instruct risky actions.

## 0.3 Tooling context
Human must provide:
- flashing tool (probe-rs/cargo-embed/vendor tool):
- interface (USB/JTAG/SWD):
- log channel (RTT/serial):
- build target triple:

---

# Phase 1: Test session setup
## 1.1 Define the test scope
The test session must declare:
- Feature or scenario: (e.g., “button debounce”, “interrupt storm handling”)
- References: `AC-xxx` (required), optional `ADR-xxx` / `API-xxx`
- Firmware build identity:
  - commit hash
  - build profile (debug/release)
  - feature flags

## 1.2 Define “signals”
Declare what will be observed:
- LED(s): color/pin, expected patterns
- Button(s): name/pin, press/hold duration
- Interrupt source: timer/GPIO/UART/DMA
- Log output: which channel, expected lines/tokens

---

# Phase 2: Structured step protocol (the heart of HiL)
Each step is a “card” with:
- Step ID
- Action (human does)
- Expected observations
- Timeout
- Pass/Fail criteria
- Evidence requirements

## Standard step card format
### HIL-<NN>: <Title>
**Action (Human):**
- …

**Expected (Observe):**
- …

**Timeout:**
- <N seconds>

**Pass criteria:**
- …

**Fail criteria:**
- …

**Evidence to capture:**
- (required) log snippet OR photo/video OR both
- if logs: include 10 lines before/after key event

---

# Phase 3: Human reporting format (mandatory)
The human must report each step in this exact format:

```text
REPORT:
step: HIL-03
result: PASS | FAIL | INCONCLUSIVE
observed:
  - <what you saw/heard>
evidence:
  - <paste logs or describe photo/video>
notes:
  - <anything weird>
````

## Rules

* If evidence is missing → result becomes `INCONCLUSIVE`.
* If the human is unsure → `INCONCLUSIVE` (not PASS).
* Any FAIL must include “what happened instead” + evidence.

---

# Phase 4: Handling ambiguity (pre-defined branches)

## 4.1 If a step fails

Do this sequence (do not improvise first):

1. Ask the human to **repeat the same step** once (same conditions).
2. If it fails again, request a **minimal reproduction**:

   * remove optional peripherals
   * use stable power source
   * reduce to smallest set of actions
3. Collect:

   * exact timing of action
   * exact LED/log behavior
   * environment notes (power, cable, OS)
4. Mark the test as FAIL and proceed to final report.

## 4.2 If results are inconsistent (flaky)

* Run the step **3 times**.
* If not consistent → label `FLAKY` and treat as a bug:

  * likely causes: bouncing, race, power instability, timing assumptions
  * require follow-up work item and additional instrumentation.

## 4.3 If the human cannot observe clearly

* Add instrumentation instead of guessing:

  * additional LED pattern
  * debug counter printed on event
  * timestamps if available
* Then rerun the same steps.

---

# Phase 5: Instrumentation rules (so humans can observe reliably)

When adding observability for HiL:

* Prefer deterministic signals:

  * event counters
  * “enter/exit” logs for state machine transitions
  * stable LED patterns (document the mapping)
* Avoid noisy spam logs (hard to interpret in real time).
* Never log secrets/keys/tokens.

---

# Phase 6: Session output — TEST report snippet (required)

At the end, produce a report snippet suitable for PR/spec mapping.

## Template (copy/paste)

```md
### TEST-XXX: Hardware HiL — <scenario name>
**Refs:** AC-xxx (required), optional ADR-xxx, API-xxx, SEC-xxx  
**Build:** <commit> | <profile> | <features> | <target>  
**Hardware:** <board/mcu> | power: <...> | wiring: <...>  
**Tooling:** <flasher> | logs: <RTT/serial>  

#### Steps
- HIL-01: <title> — PASS/FAIL/INCONCLUSIVE — evidence: <link/paste>
- HIL-02: <title> — PASS/FAIL/INCONCLUSIVE — evidence: <link/paste>
- …

#### Summary
- Overall: PASS / FAIL / INCONCLUSIVE / FLAKY
- Notes: <key observations + anomalies>
- Follow-ups: <bugs/instrumentation needed>
```

## Minimum bar for “PASS”

* All required steps are PASS
* No INCONCLUSIVE steps
* Evidence included for at least the key transitions
* If any flakiness observed → not PASS (label FLAKY)

---

# Quick-start: a generic HiL sequence (starter kit)

Use this as a default step list, then customize.

### HIL-01: Flash and boot

Action: flash firmware, power-cycle
Expected: boot LED pattern + “boot ok” log line
Timeout: 30s
Evidence: boot logs

### HIL-02: Human action gate (button press)

Action: press/hold button for X seconds
Expected: log “button down/up” + LED change
Timeout: 10s
Evidence: log snippet around event

### HIL-03: Interrupt trigger

Action: trigger interrupt source (GPIO toggle / timer start)
Expected: counter increments; no crash; stable state transitions
Timeout: 10s
Evidence: log snippet + counter value

### HIL-04: Long-run stability (optional)

Action: let it run for N minutes
Expected: no reset; no watchdog; no abnormal patterns
Timeout: N minutes
Evidence: periodic log heartbeat

````

---

## 2) Command: `/hil` (ตัวช่วยเริ่ม session ให้เร็ว)

วางไฟล์นี้ที่: `.opencode/commands/hil.md` :contentReference[oaicite:3]{index=3}

```markdown
---
description: Start a human-in-the-loop hardware test session and produce a TEST-xxx report mapped to AC-xxx.
agent: build
---

We are running a Human-in-the-loop hardware test session for: $ARGUMENTS

Repo context:
- branch: !`git branch --show-current`
- commit: !`git rev-parse --short HEAD`
- changed files: !`git diff --name-only`

Instructions:
1) Load skills: hardware-hil-testing, firmware-testing, team-contract-ids.
2) Ask the user for preflight info (board/mcu, power, wiring, flasher, log channel).
3) Ask for the target references (AC-xxx required; optional ADR/API/SEC).
4) Produce a timeboxed step-by-step runbook (HIL-01..).
5) Collect user REPORT blocks per step.
6) Output a final TEST-xxx snippet suitable for PR/spec traceability.
