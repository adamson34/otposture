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
subsystem: "SS-06"
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

# Behavioral Contract BC-6.06.003: Offline Guarantee

## Description

The binary performs **zero network operations** under every command and condition — no telemetry, no update checks, no DNS, no sockets (DI-007). This is a verifiable product boundary, not a configuration default. It is also the differentiator OT buyers can audit.

## Preconditions

1. Any invocation, any flags, any environment.

## Postconditions

1. Process completes having made no network syscalls (socket/connect/send/DNS), observable under syscall tracing on every supported platform.
2. Behavior is identical with networking disabled at the OS level (air-gap parity).

## Invariants

1. The dependency tree contains no networking crates in the binary's feature graph (CI-enforced: `cargo deny`/feature audit).
2. No code path constructs URLs for fetching; documentation links in help text are inert strings.
3. Future features must preserve this — any networking proposal is a major-version product decision, not a patch (brief: "hard product boundary... ever").

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Catalog/report paths on network shares (SMB/NFS) | Filesystem I/O through the OS is fine — the contract bars network *protocol* use by the process, not networked filesystems mounted by the OS |
| EC-002 | Proxy environment variables set (HTTP_PROXY etc.) | Ignored — nothing consults them |
| EC-003 | `--version` / crash reporting | Prints local info only; no phone-home of any kind |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Full command suite under `strace`/dtrace network filter | Zero network syscalls | happy-path |
| Same suite with all interfaces down | Identical outputs/exit codes | happy-path |
| Dependency audit in CI | No network-capable crates in binary features | happy-path |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-083 | Zero network syscalls across the integration suite (traced) | CI syscall-trace harness |
| VP-084 | Dependency graph excludes networking crates | CI cargo-deny policy |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | All (cross-cutting) |
| L2 Domain Invariants | DI-007 |
| Differentiator | "Offline, no-footprint, data-sovereign" |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-5.05.003 — composes with (report has no external resources)
