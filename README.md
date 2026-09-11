# cloud-itonami-isco-7542

Open Occupation Blueprint for **ISCO-08 7542**: Shotfirers and Blasters.

This repository designs a forkable OSS business for a blast-operation scheduling/logistics coordination service: a blast-operation scheduling/logistics coordination robot manages blast-log/progress record logging, crew/site scheduling, safety-concern flagging and blasting-equipment/administrative-supply order coordination under a governor-gated actor, so the blasting operator keeps its own operating records instead of renting a closed blast-scheduling SaaS.

**This actor coordinates BLAST-OPERATION SCHEDULING/LOGISTICS ONLY — it never handles explosives itself and never makes a blast-authorization or blast-execution decision.** Shotfirers and blasters handle explosives for controlled demolition, mining and quarrying blasting operations — errors can cause death, serious injury or major property destruction, categorically higher-stakes than ordinary workshop trades, on par with aviation and diving in this catalog. The actor's closed op-allowlist contains no op that directly finalizes a blast-authorization decision (approving a detonation to proceed) or a blast-execution decision, nor overrides a blasting supervisor's/site-safety-officer's judgment. Any proposal that attempts any of these is a hard, permanent block, never overridable by human approval, and NEVER auto-commit-eligible under any confidence level.

**Maturity: `:implemented`.** `src/blastcoord/` implements the
`BlastCoordActor` as a `langgraph.graph/state-graph`
(`blastcoord.actor`) wired to a `Blast-Operation Scheduling &
Logistics Coordination Advisor` (`blastcoord.advisor`) and an
independent `BlastCoordGovernor` (`blastcoord.governor`), following
the itonami actor pattern (ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok? true) +-> :request-approval (:escalate? true, human-in-the-loop
interrupt) +-> :hold (:hard? true)`. See `kbb -M:test` output for
the current test/assertion counts.

HARD invariants (always `:hold`, never overridable): the blaster/site
record must be independently verified/registered before any action;
a referenced blaster must be a registered certified crew member
belonging to that site; `:effect` must be `:propose` only (no hardware
dispatch, no explosives handling); the closed op-allowlist is enforced
(no op in the allowlist finalizes a blast-authorization decision,
finalizes a blast-execution decision, or overrides blasting-
supervisor/site-safety-officer authority); and any proposal that
attempts to directly finalize a blast-authorization decision (approving
a detonation to proceed), finalize a blast-execution decision, or
override a blasting supervisor's/site-safety-officer's judgment is a
hard, **permanent** block — detected as finalization/execution action
phrases (never bare nouns like "explosive"/"blast"/"detonation"/
"shot"/"round", which are ordinary vocabulary for this domain and must
not false-trip the guard).

Always-escalate ops (human sign-off regardless of confidence, mapping
this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (every surfaced misfire, exclusion-zone-breach
or explosive-storage-condition concern, ALWAYS, no exceptions, ever)
and `:coordinate-supply-order` above the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a blast-operation scheduling/logistics coordination robot performs blast-log/progress record logging, crew/site-schedule proposals, safety-concern surfacing and blasting-equipment/administrative-supply order coordination under an actor that proposes
actions and an independent **Blast-Operation Scheduling & Logistics Coordination Governor** that gates them. The governor never
dispatches hardware itself, never handles explosives, never finalizes a blast-authorization or blast-execution decision, and never overrides a blasting supervisor's/site-safety-officer's judgment; `:high`/`:safety-critical` actions (such as a safety-concern flag or an above-threshold supply order) require human sign-off.

## Core Contract

```text
site roster + blaster roster + site schedule
        |
        v
Blast-Operation Scheduling & Logistics Coordination Advisor -> BlastCoordGovernor -> log record/schedule/order, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a blast-authorization decision (approving a detonation to
proceed), finalize a blast-execution decision, override a blasting
supervisor's/site-safety-officer's judgment, suppress an operating
record, or disclose sensitive data without governor approval and audit
evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7542`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
