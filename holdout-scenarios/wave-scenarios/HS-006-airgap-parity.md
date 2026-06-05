---
document_type: holdout-scenario
level: ops
version: "1.0"
status: draft
producer: product-owner
timestamp: 2026-06-05T15:44:03
inputs: [stories/, behavioral-contracts/, prd.md]
input-hash: "56ca7a142db7b60de70a3281283396cb"
traces_to: ""
id: "HS-006"
category: "security-probes"
must_pass: "true"
priority: "must-pass"
epic_id: "E-6"
behavioral_contracts: [BC-6.06.003, BC-5.05.003]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: null
---

# Holdout Scenario: The Air-Gapped Plant Gets Exactly the Same Product

> NEVER show to implementer/test-writer agents.

## Scenario

1. The full command lifecycle (init → assess → snapshot → status → diff → report → verify) runs twice: once on a networked host under syscall tracing, once with all interfaces down and hostile proxy env vars set (HTTP_PROXY pointing at a listener).
2. Both runs use identical inputs.
3. Zero network syscalls observed in run one; the proxy listener receives zero connections; both runs produce byte-identical outputs and exit codes; the generated HTML report opened from file:// makes no external requests.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-6.06.003 | postconditions 1–2, EC-002 | trace + airgap parity + proxy ignorance |
| BC-5.05.003 | postcondition 2 | report self-containment |

## Verification Approach

strace/dtrace filter on socket/connect/sendto/DNS; netcat listener as fake proxy; diff the two runs' stdout/stderr/exit codes/artifacts; HTML resource audit (no http(s):// fetchable references).

## Evaluation Rubric

Functional correctness 0.5 (zero syscalls, zero connections); data integrity 0.3 (byte parity); edge handling 0.2 (proxy vars, report file:// behavior).

## Edge Conditions

Store and catalog on an SMB-mounted path (networked filesystem via OS is permitted — the process itself must still open no sockets).

## Failure Guidance

"HOLDOUT LOW: HS-006 (satisfaction: 0.XX) -- the binary attempted network activity or behaved differently when offline."
