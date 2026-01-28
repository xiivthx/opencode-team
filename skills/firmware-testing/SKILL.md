---
name: firmware-testing
description: Embedded Rust testing strategy focusing on host-based simulation and safe on-target integration.
compatibility: opencode
metadata:
  owner: alex
  domain: firmware
  freedom: low
---

# Firmware Testing (Alex)

This skill standardizes how we test embedded Rust firmware without relying on a board for every change.

## When to use me
Use this skill when:
- Writing or changing drivers (I2C/SPI/GPIO), state machines, protocol code
- Refactoring firmware architecture to improve testability
- Adding `unsafe`, DMA, interrupt-driven code, or FFI
- Setting up CI checks for firmware crates

## Goal
Maximize **host-based confidence** (fast, deterministic CI tests) and use **on-target tests** for integration-only coverage.

## Related skills (routing)
- For on-board human-in-the-loop tests: skill({ name: "hardware-hil-testing" })
- For general TDD rules and host testing logic: skill({ name: "tdd-playbook" })
- For traceability and AC/TEST mapping: skill({ name: "team-contract-ids" })
- For quality gates and determinism: skill({ name: "qa-gates" })

## Recommended Crates (Embedded)
- **`embassy`**: Modern embedded framework and async-await for microcontrollers.

---

# 1) Architecture for testability (non-negotiable)
## 1.1 Separate “pure logic” from hardware glue
Split firmware into layers:

1) **Core logic crate/module** (pure, testable)
- No register poking
- Minimal `unsafe` (ideally none)
- Deterministic state machines and parsing

2) **HAL adapter layer** (thin glue)
- Owns `embedded-hal` traits / concrete HAL types
- Converts hardware events → core logic inputs
- Contains the “messy” platform-specific bits

3) **Board/runtime integration**
- Startup, interrupts, DMA setup, peripherals init
- Logging/tracing framework integration

## 1.2 Prefer dependency injection
Drivers should accept trait objects/generics over `embedded-hal` traits (I2C/SPI/GPIO/Delay).
This allows host tests using mock implementations.

---

# 2) Host-based tests (default path)
## 2.1 Use embedded-hal mock implementations
Do NOT rely solely on hardware-in-loop.
Write unit tests that instantiate drivers with mocked `embedded-hal` implementations.

### What to test on host
- State transitions
- Protocol framing/parsing
- Error handling (NACK/timeouts/invalid data)
- Retry/backoff behavior (if any)
- Boundary values and invariants

### Minimal testing pattern (conceptual)
- Arrange expected bus transactions
- Run driver method
- Assert output and ensure all expectations were consumed

```rust
#[test]
fn writes_config_then_reads_status() {
    // Pseudocode: adapt to your embedded-hal version and mock crate.
    // 1) Create mock bus with expected transactions
    // 2) Create driver with mock bus
    // 3) Call driver function
    // 4) Assert result and verify mock expectations
}
````

## 2.2 Determinism rules (firmware edition)

* No timing sleeps in host tests
* No randomness unless seeded
* No test ordering dependencies
* No “real” hardware required for CI

---

# 3) On-target tests (optional, integration-only)

Host tests catch 80–95% of logic failures. Use on-target tests for:

* Interrupt/DMA correctness
* Peripheral timing behavior
* RTT/defmt logging integration
* Real sensor/actuator interactions

## 3.1 Keep on-target tests minimal

* A small smoke suite (boot, peripheral init, one read/write per bus)
* A single “golden path” end-to-end scenario if needed
* Timebox and keep runtime short

## 3.2 Tooling expectations (project-dependent)

* Use probe-based workflows for flashing and RTT output when applicable.
* Prefer a consistent runner and config checked into repo.

---

# 4) Unsafe policy (strict)

## 4.1 Minimize unsafe and wrap it

* Keep `unsafe` blocks as small as possible
* Hide unsafe behind safe abstractions
* Prefer compile-time proofs (types) over comments where possible

## 4.2 SAFETY comments are mandatory

Every `unsafe` block must have a `// SAFETY:` comment that states:

* which invariants are relied on
* who upholds them (caller/module/system)
* why the operations are safe under those invariants

Example format:

```rust
// SAFETY: DMA buffer is exclusively owned for the duration of the transfer,
// aligned as required by the peripheral, and not accessed from interrupts.
unsafe { /* ... */ }
```

## 4.3 DMA + interrupts checklist

Before merging code that uses DMA/interrupts:

* Prove exclusive ownership of buffers
* Prove interrupt preemption does not violate aliasing/lifetimes
* Confirm memory ordering assumptions (if any) are valid
* Document required critical sections

---

# 5) CI expectations (firmware)

Coordinate with CI owner to ensure:

* Host tests run on x86_64 for core logic
* Firmware builds run for target triples (cross compilation)
* Optional: minimal on-target smoke (only if infra exists)

Recommended CI sequence for firmware crates:

* `cargo fmt --check`
* `cargo clippy -- -D warnings`
* `cargo test` (host, logic crate/modules)
* `cargo build --target <mcu-target>` (firmware build)

---

# 6) Hardware-in-loop (HIL) rule

HIL is for:

* final integration checks
* confirming electrical/timing reality
  Not for:
* validating state machines or driver logic that can be simulated

If HIL is required:

* record board/firmware version
* record steps and expected output
* keep a minimal reproducible test plan

---

# Pre-merge checklist (firmware)

* [ ] Core logic is testable on host (unit tests exist)
* [ ] HAL glue is thin and isolated
* [ ] Mock-based tests cover key state transitions and error paths
* [ ] No timing/randomness/order dependency in tests
* [ ] All `unsafe` blocks have `// SAFETY:` comments with explicit invariants
* [ ] DMA/interrupt code has an ownership + preemption story
* [ ] Target builds succeed for required MCU targets
