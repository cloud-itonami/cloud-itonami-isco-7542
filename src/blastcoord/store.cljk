(ns blastcoord.store
  "SSoT for the ISCO-08 7542 shotfirers and blasters blast-operation
  scheduling/logistics coordination actor (itonami actor pattern,
  ADR-2607121000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — a blast-operation scheduling/logistics coordination robot
  proposes crew/site scheduling, blast-log/progress record logging,
  safety-concern flags and blasting-equipment/administrative-supply
  order coordination under this advisor/governor pair, which never
  dispatches hardware itself, never handles explosives, and never
  finalizes a blast-authorization decision, a blast-execution
  decision, or overrides a blasting supervisor's/site-safety-officer's
  judgment). Modeled on cloud-itonami-isco-7232's aerocoord.store.

  Domain:

    site     — a registered blast site/operation under coordination
               (:site-id, :name, :location).
    blaster  — a registered certified shotfirer/blaster
               {:blaster-id :site-id :name :role}, belonging to
               exactly one registered site (the site currently
               assigned to this blaster for this operation).
    record   — a committed operating record (a logged blast-log/
               progress record, scheduling proposal, safety-concern
               flag or supply-order coordination entry) — written ONLY
               via commit-record!. This actor coordinates blast-
               operation scheduling/logistics ONLY — a `record` is a
               coordination artifact, never explosives handling, a
               blast-authorization decision, a blast-execution
               decision, or a blasting-supervisor's-/site-safety-
               officer's-judgment override.
    ledger   — append-only audit trail, commit or hold.")

(defprotocol Store
  (site [s site-id])
  (blaster [s blaster-id])
  (records-of [s site-id])
  (ledger [s])
  (register-site! [s st])
  (register-blaster! [s b])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (site [_ site-id] (get-in @a [:sites site-id]))
  (blaster [_ blaster-id] (get-in @a [:blasters blaster-id]))
  (records-of [_ site-id] (filter #(= site-id (:site-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-site! [s st]
    (swap! a assoc-in [:sites (:site-id st)] st) s)
  (register-blaster! [s b]
    (swap! a assoc-in [:blasters (:blaster-id b)] b) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:sites {} :blasters {} :records [] :ledger []}
                                   seed)))))
