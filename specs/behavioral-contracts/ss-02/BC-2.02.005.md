---
document_type: behavioral-contract
level: L3
version: "1.1"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [domain-spec/L2-INDEX.md, product-brief.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: domain-spec/L2-INDEX.md
origin: greenfield
subsystem: "SS-02"
capability: "CAP-002"
lifecycle_status: active
introduced: v0.1.0
modified: []
deprecated: null
deprecated_by: null
replacement: null
retired: null
removed: null
removal_reason: null
---

# Behavioral Contract BC-2.02.005: Single-Writer Locking

## Description

Mutating commands acquire an advisory lock on the store; a concurrent second writer fails fast with a clear message rather than corrupting or losing updates (FM-006). Read-only commands never block.

## Preconditions

1. Store exists; a mutating command (assess/answer/override/snapshot/migrate/upgrade) is invoked.

## Postconditions

1. Lock acquired: a lockfile records PID, hostname, and start time; released on exit (including signal-handled termination paths).
2. Lock held by a live process: E-LOCK-001 "store in use by PID N since T"; exit non-zero; no waiting by default.
3. Stale lock (PID dead on same host, or age > threshold): announced and broken automatically; operation proceeds.

## Invariants

1. Read-only commands (status, diff, report, verify) never take the lock and never fail due to it.
2. Lock contents contain no personal data — PID/host/time only (DI-013).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Lockfile from another host (store on a shared drive) | PID liveness unknowable: honor lock until age threshold; message explains the shared-drive caveat |
| EC-002 | Process killed -9, lock left behind | Next mutating command detects dead PID, breaks lock with notice |
| EC-003 | Two processes race the lock creation itself | OS-level atomic create (O_EXCL) decides; exactly one wins |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Second `assess` while first runs | E-LOCK-001, exit ≠ 0, first session unaffected | error |
| `status` while `assess` holds the lock | Succeeds (reads last durable state) | happy-path |
| Mutating command after kill -9 of prior holder | Stale lock broken with notice; proceeds | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-018 | Under concurrent mutation attempts, at most one writer proceeds | proptest (process-level test) |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-002 |
| L2 Failure Modes | FM-006 |
| L2 Assumptions | ASM-008 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.002 — composes with (lock guards the atomic-write window)
