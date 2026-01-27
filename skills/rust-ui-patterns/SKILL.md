---
name: rust-ui-patterns
description: Rust UI playbook for consistent async states, error UX mapping from backend error codes, and maintainable state management (avoid prop-drilling). Use when building or refactoring UI components/screens, especially those calling APIs.
compatibility: opencode
metadata:
  owner: lyra
  domain: ui
  freedom: medium
---

# Rust UI Patterns (Lyra)

This skill standardizes UI behavior around:
- **Async state handling** (loading/empty/success/error/retry)
- **Error UX** (mapping backend error codes to user-safe messages and actions)
- **State management discipline** (local vs global; avoid prop-drilling)
- **Accessibility baseline** (keyboard + focus + labels)

## When to use me
Use this skill when:
- You build/modify screens that call backend APIs
- You introduce a new component with interactive states
- You handle auth/session state
- You need consistent error and loading behavior across the app

## Core principle
The UI must be **predictable** under failure:
- no spinning forever
- no raw internal error text shown to users
- clear recovery actions (retry, re-auth, edit input)

## Related skills (routing)
- For tokens/classes/state styling rules: use skill({ name: "tailwind-design-system" }).
- For cross-team design heuristics (complexity budget, coupling): use skill({ name: "engineering-principles" }).
- For source-of-truth IDs and traceability: use skill({ name: "team-contract-ids" }).

---

# 1) Async state model (canonical)
Represent remote data with a small state machine. Use a single enum-like model across screens:

- `Idle` (optional): not started yet
- `Loading`
- `Ready(data)`
- `Empty` (when applicable)
- `Error(ui_error)` (always user-safe)

## Rules
- Every API call must transition through `Loading → Ready|Empty|Error`
- “Empty” is not the same as “Error”
- “Retry” must be possible for retryable failures
- Never keep multiple boolean flags (`isLoading`, `hasError`, `isEmpty`) if they can contradict.

### Recommended structure
If your UI framework supports enums, use them. If not, emulate with tagged structs.

```rust
pub enum Remote<T> {
    Idle,
    Loading,
    Ready(T),
    Empty,
    Error(UiError),
}
````

---

# 2) Error UX mapping (backend → UI)

Backend errors must follow a stable error contract (`error.code`, `retryable`) produced by backend standards.
UI must map codes to:

* user-facing message
* severity (banner/toast/inline)
* recovery action (retry, login, edit input)

## Never show raw backend errors

* Do not display stack traces, SQL errors, internal messages, or debug dumps.
* Only show curated messages or safe backend-provided messages if policy allows.

## Canonical UiError shape

```rust
pub struct UiError {
    pub code: String,
    pub message: String,
    pub retryable: bool,
    pub action: UiErrorAction,
}

pub enum UiErrorAction {
    None,
    Retry,
    Reauthenticate,
    EditInput,
    ContactSupport,
}
```

## Mapping table (example; adapt to your codes)

| Backend `error.code` | UI Message                                          | Action         | Presentation          |
| -------------------- | --------------------------------------------------- | -------------- | --------------------- |
| `INVALID_ARGUMENT`   | “Please check your input.”                          | EditInput      | Inline near field     |
| `UNAUTHORIZED`       | “Your session expired. Please sign in again.”       | Reauthenticate | Banner                |
| `FORBIDDEN`          | “You don’t have access to do that.”                 | None           | Banner                |
| `NOT_FOUND`          | “We couldn’t find that item.”                       | None           | Inline or Empty state |
| `RESOURCE_CONFLICT`  | “Someone else changed this. Refresh and try again.” | Retry          | Banner + Retry        |
| `RATE_LIMITED`       | “Too many requests. Try again in a moment.”         | Retry          | Toast                 |
| `UPSTREAM_TIMEOUT`   | “Network timed out. Try again.”                     | Retry          | Banner + Retry        |
| `INTERNAL`           | “Something went wrong. Try again.”                  | Retry          | Banner                |

> Keep your mapping in one module (single source of truth):
>
> * `ui/error_mapping.rs` (or equivalent)
> * Make it easy to update without hunting across screens.

## Error presentation rules

* Form validation errors: inline by field + summary at top if multiple
* Screen-level failure: banner with Retry if retryable
* Background refresh failure: toast (do not wipe already-shown data)
* Empty results: show Empty state, not Error

---

# 3) Component state exhaustion (required)

Every interactive component must define styles/behavior for:

* `DEFAULT`
* `HOVER/FOCUS` (keyboard visible)
* `DISABLED`
* `LOADING` (spinner/skeleton)
* `ERROR` (validation / server failure indicator)

> If design tokens are used, coordinate with `tailwind-design-system`.

---

# 4) State management strategy (avoid prop-drilling)

## Decision rule

* Use **local state** for: toggles, input values, component-only interactions
* Use **global state** for: session/auth token, user identity, feature flags, shared caches, cross-page filters

## Anti-pattern: deep prop drilling

* Avoid passing state/handlers down more than 2 levels.
* Use your framework’s context/signal/store mechanism.

## Session/auth must be global

* Tokens/session must not be threaded through props.
* Auth expiry triggers a global transition (sign-in required) and a consistent UI message.

---

# 5) Networking & retry behavior

## Retry rules

* Only auto-retry if explicitly safe (idempotent GETs, retryable errors).
* For non-idempotent operations (POST/PUT), prefer **user-triggered retry**.

## Loading behavior

* Initial load: skeleton/placeholder, not blank screen
* Refresh: keep old data + small “refreshing” indicator (avoid jarring resets)

## Timeouts

* If request times out and backend marks `retryable: true`, show Retry CTA.

---

# 6) Accessibility baseline (minimum)

* Keyboard navigation works for all controls
* Focus ring visible and not suppressed
* Inputs have labels and error descriptions
* Buttons have accessible names (icon buttons require aria-label)
* Do not use color as the only signal (error state needs icon/text)

---

# 7) Testing expectations (UI)

Minimum tests per screen/component change:

* Snapshot for default + loading + error (if your stack supports it)
* Unit tests for error mapping (code → UiError)
* Interaction tests for retry and auth expiry flows when feasible

---

# Pre-merge checklist (UI)

* [ ] Async states implemented as a single coherent model
* [ ] Error mapping centralized and user-safe
* [ ] No raw backend/internal errors shown to users
* [ ] Retry and recovery actions are present where applicable
* [ ] Component states defined (default/hover/focus/disabled/loading/error)
* [ ] Auth/session handled globally
* [ ] A11y baseline met (keyboard + focus + labels)
