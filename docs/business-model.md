# Business Model: Blast-Operation Scheduling & Logistics Coordination Service

## Classification

- Repository: `cloud-itonami-isco-7542`
- ISCO-08: `7542`
- Occupation: Shotfirers and Blasters
- Social impact: blast-safety, worker-safety, public-safety

## Scope

**This actor coordinates blast-operation scheduling and logistics
only.** It never handles explosives itself, never finalizes a
blast-authorization decision (approving a detonation to proceed),
never finalizes a blast-execution decision, and never overrides a
blasting supervisor's or site-safety-officer's judgment. Shotfirers
and blasters handle explosives for controlled demolition, mining and
quarrying blasting operations — errors can cause death, serious
injury or major property destruction, categorically higher-stakes
than ordinary workshop trades — so every proposal this actor's
advisor can make is limited to coordination, not execution and not
authorization.

## Customer

- mining and quarrying blasting operators
- demolition and construction blasting crews and crew leads

## Offer

- blast-log/progress record logging (task, round reference, materials
  usage, progress)
- crew/site-schedule scheduling proposals
- safety-concern surfacing (misfire, exclusion-zone breach,
  explosive-storage condition)
- blasting-equipment/administrative-supply order coordination (NOT
  explosives themselves — explosives procurement/handling is entirely
  out of scope for this administrative-coordination actor)

## Revenue

- monthly coordination-platform retainer
- per-site logistics fee

## Trust Controls

- no blast-authorization decision (approving a detonation to proceed)
  is ever finalized by this actor
- no blast-execution decision is ever finalized by this actor
- no blasting supervisor's or site-safety-officer's judgment is ever
  overridden by this actor
- every safety-concern flag ALWAYS escalates to human sign-off, no
  exceptions, ever
- supply orders above the registered cost threshold always escalate to
  human sign-off
- site and blaster provenance is independently verified before any
  coordination action
- coordination and audit records are auditable, not editable
