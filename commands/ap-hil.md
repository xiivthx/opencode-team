---
description: Hardware Human-in-the-loop test protocol: step-by-step prompts, user observations, and a repeatable test log.
subtask: true
---

## Skills
- firmware-testing
- qa-gates (determinism principles adapted to hardware)
- engineering-principles (simplicity, evidence-first)

Intent: $ARGUMENTS

## Safety rules
- Never instruct anything unsafe (mains voltage, shorting power rails, bypassing protections).
- Ask for explicit confirmation before any action that could damage hardware.
- Default to low-risk observation first (LEDs/serial logs).

## Create a test log (write file)
Create: `.opencode/hil/{YYYYMMDD}-{slug}.md`
If directory missing, create `.opencode/hil/`.

## Protocol
### 1) Establish the test contract (you write)
- Device/board name (unknown -> ask once)
- Firmware build target
- Expected signals: LED patterns, UART logs, button events, interrupts
- Pass/fail criteria (observable)

### 2) Run steps (interactive)
For each step:
1) You provide:
   - what to do physically (press button, toggle switch, induce interrupt)
   - what to observe (LED blink count, log line substring, timing bounds)
2) User replies with observations.
3) You record result in the log and decide:
   - PASS / FAIL / RETRY (with changed hypothesis)

### 3) Fault isolation (when failing)
- Propose the minimum set of experiments to isolate:
  - input path (GPIO)
  - timing/interrupt
  - state machine transitions
  - peripheral comms (I2C/SPI)

### 4) Output
- Updated test log path
- Current verdict (PASS/FAIL/BLOCKED)
- Next single best step (exact instruction to user)
