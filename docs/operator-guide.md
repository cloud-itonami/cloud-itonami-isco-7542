# Operator Guide

## First Deployment

1. Define the operator's site roster and blaster-certification/registration process.
2. Define consent and purpose categories for logged blast-log/progress records.
3. Run synthetic coordination cases (blast-log/progress record logging, scheduling, safety-concern flags, supply orders).
4. Enable human-reviewed sign-off for `:high`/`:safety-critical` actions — every safety-concern flag, no exceptions ever, and every above-threshold supply order.
5. Measure coordination outcomes and audit coverage.

## Minimum Production Controls

- consent and disclosure log
- safety-critical escalation path (misfire, exclusion-zone-breach and explosive-storage-condition concerns ALWAYS reach a human — no exceptions, ever)
- provenance for all sites and blasters before any coordination action
- human review for high-risk cases
- audit export for all gated actions

## Scope Boundary (Mandatory)

This actor coordinates blast-operation scheduling and logistics ONLY.
Operators must not wire this actor's output into any system that
would let it directly finalize a blast-authorization decision
(approving a detonation to proceed), finalize a blast-execution
decision, or override a blasting supervisor's or site-safety-officer's
judgment — the governor's closed op-allowlist and scope-exclusion rule
are the last line of defense, not the only one; operator-side
integrations must not create a path around them. This actor never
handles explosives itself: `:coordinate-supply-order` covers
blasting-equipment/administrative-supply procurement only, never
explosives themselves — explosives procurement/handling is entirely
out of scope for this administrative-coordination actor.

## Certification

Certified operators must prove that the governor gates every
safety-critical robot action, that safety-critical risks escalate to
humans unconditionally, and that no integration allows this actor to
finalize a blast-authorization decision, finalize a blast-execution
decision, or override blasting-supervisor/site-safety-officer
authority.
