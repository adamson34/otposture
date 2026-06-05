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
id: "HS-007"
category: "security-probes"
must_pass: "true"
priority: "must-pass"
epic_id: "E-5"
behavioral_contracts: [BC-5.05.003, BC-5.05.004]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: null
---

# Holdout Scenario: The Exec Report Survives Hostile Free Text and Careless Re-Runs

> NEVER show to implementer/test-writer agents.

## Scenario

1. A store whose site name, answer notes, override rationales, and snapshot labels all contain HTML/JS payloads (`<script>`, `<img onerror=…>`, broken entities, RTL overrides) and multi-KB unicode.
2. The user generates `report --exec`, then re-runs the same command, then re-runs with `--force`, then targets the store directory itself as output.
3. The HTML renders all payloads as inert text (parsed DOM contains no injected elements); second run refuses E-RPT-001 leaving the original intact; `--force` replaces atomically; store-directory target refuses E-RPT-003 even with --force.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-5.05.003 | invariant 3, EC-004 | escaping of all store-sourced strings |
| BC-5.05.004 | postconditions 2–3, EC-003 | overwrite + placement safety |

## Verification Approach

Inject payload corpus via normal commands; generate the report; parse with an HTML parser and assert no script/element injection; check disclosure sections still render (overrides with hostile rationales); run the overwrite sequence and inspect file states between steps.

## Evaluation Rubric

Functional correctness 0.4 (zero injection); error quality 0.2 (E-RPT remedies); edge handling 0.2 (unicode/RTL); data integrity 0.2 (atomic replace, no truncated artifact at any step).

## Edge Conditions

Disk-full during report write (pre-existing report intact); payload in the floor-gap goal title path.

## Failure Guidance

"HOLDOUT LOW: HS-007 (satisfaction: 0.XX) -- store content reached the report unescaped, or output-path safety rules failed."
