(ns blastcoord.governor
  "BlastCoordGovernor — the independent safety/traceability layer named
  in this repository's README/business-model.md, gating every
  blast-operation scheduling/logistics coordination proposal an
  advisor may make for a blast site under coordination. The governor
  never dispatches hardware itself, never handles explosives, and
  never allows a proposal to finalize a blast-authorization decision
  (approving a detonation to proceed), finalize a blast-execution
  decision, or override a blasting supervisor's/site-safety-officer's
  judgment — this actor coordinates BLAST-OPERATION SCHEDULING/
  LOGISTICS ONLY. Modeled on cloud-itonami-isco-7232's
  aerocoord.governor.

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. site provenance         — the blaster/site record must be
                                 independently verified/registered
                                 before any action.
    2. no-actuation            — proposal :effect must be :propose
                                 (the governor never dispatches
                                 hardware and never handles explosives;
                                 it only gates what the advisor may
                                 coordinate).
    3. closed op-allowlist     — :op must be one of the four
                                 coordination ops (:log-work-record,
                                 :schedule-crew-operation,
                                 :flag-safety-concern,
                                 :coordinate-supply-order). No op that
                                 directly finalizes a blast-
                                 authorization decision, finalizes a
                                 blast-execution decision, or overrides
                                 blasting-supervisor/site-safety-
                                 officer authority exists in this
                                 allowlist — these decision classes are
                                 structurally absent, not merely
                                 gated. This actor never handles
                                 explosives itself; explosives
                                 procurement/handling is entirely
                                 out of scope for this administrative-
                                 coordination actor (:coordinate-
                                 supply-order covers blasting-
                                 equipment/administrative-supply
                                 procurement only, never explosives
                                 themselves).
    4. site-mismatch           — if the proposal names a site, it must
                                 be the SAME site verified for this
                                 request (defense-in-depth against a
                                 proposal quietly targeting a
                                 different, unverified site).
    5. blaster basis           — if the proposal references a blaster,
                                 that blaster must be a REGISTERED
                                 certified shotfirer/blaster belonging
                                 to this site (an unregistered or
                                 foreign-site blaster reference is not
                                 a routine scheduling proposal).
    6. scope-exclusion         — a proposal that attempts to finalize
                                 a blast-authorization decision
                                 (approving a detonation to proceed),
                                 to finalize a blast-execution
                                 decision, or to override a blasting
                                 supervisor's or site-safety-officer's
                                 judgment, is a hard, PERMANENT block —
                                 never overridable by human approval,
                                 regardless of confidence or stake, and
                                 NEVER auto-commit-eligible under any
                                 confidence level. Detected as
                                 finalization/execution ACTION PHRASES
                                 (e.g. 'authorize the blast to
                                 proceed', 'finalize the blast
                                 operation', 'override the blasting
                                 supervisor's judgment') in free-text
                                 proposal fields, never as bare domain
                                 nouns ('explosive', 'blast',
                                 'detonation', 'shot', 'round') — bare-
                                 noun matching would false-trip on the
                                 default mock advisor's own routine
                                 rationale text, since this actor's
                                 entire domain is blast-operation
                                 coordination. See
                                 `blastcoord.governor-test`
                                 `default-mock-advisor-proposals-never-self-trip-scope-exclusion`.

  ESCALATION invariants (:escalate? true, ALWAYS human sign-off,
  regardless of confidence):
    7. :op :flag-safety-concern always escalates (a surfaced misfire,
                                 exclusion-zone breach or explosive-
                                 storage-condition concern ALWAYS
                                 requires human review — the governor
                                 never resolves a safety concern
                                 itself, and this is unconditional —
                                 no confidence-level exception, ever).
    8. :op :coordinate-supply-order with :cost above
                                 `supply-order-cost-threshold` always
                                 escalates.
    9. low confidence (< `confidence-floor`)."
  (:require [kotoba.lang.text :as str]
            [blastcoord.store :as store]))

(def confidence-floor 0.6)

(def supply-order-cost-threshold 20000)

(def ^:private allowed-ops
  #{:log-work-record :schedule-crew-operation :flag-safety-concern
    :coordinate-supply-order})

(def ^:private always-escalate-ops #{:flag-safety-concern})

;; Scope-exclusion is matched as finalization/execution ACTION
;; PHRASES, never as bare nouns ("explosive", "blast", "detonation",
;; "shot", "round") — this actor's entire domain is blast-operation
;; coordination, so bare-noun matching would false-trip on the default
;; mock advisor's own routine rationale text (e.g. "proposed
;; :coordinate-supply-order for site BS-1" naming blasting mats and
;; signage, or a crew-schedule proposal naming a round under
;; preparation). See governor-test's dedicated self-trip guard.
(def ^:private scope-exclusion-phrases
  ["authorize the blast to proceed"
   "authorize the detonation to proceed"
   "approve the blast to proceed"
   "approve the detonation to proceed"
   "clear the blast to proceed"
   "clear the site for detonation"
   "finalize the blast authorization"
   "finalize the blast-authorization decision"
   "finalize the detonation authorization"
   "finalize the blast operation"
   "finalize the blast execution"
   "finalize the blast-execution decision"
   "execute the blast directly"
   "execute the detonation directly"
   "perform the blast directly"
   "perform the detonation directly"
   "initiate the detonation directly"
   "trigger the detonation directly"
   "fire the shot directly"
   "dispatch the crew to detonate"
   "dispatch the shotfirer to fire the shot"
   "sign off the blast authorization"
   "sign the blast authorization"
   "issue the blast authorization"
   "override the blasting supervisor's judgment"
   "override the blasting supervisor"
   "override the site safety officer's judgment"
   "override the site safety officer"
   "bypass the blasting supervisor"
   "bypass the site safety officer"
   "bypass the exclusion zone clearance"])

(defn- scope-excluded-text [proposal]
  (str/lower (str (:rationale proposal) " " (:description proposal))))

(defn scope-exclusion-violation?
  "true if any free-text field of `proposal` contains a
  finalization/execution action phrase attempting to finalize a
  blast-authorization decision, finalize a blast-execution decision,
  or override blasting-supervisor/site-safety-officer authority.
  Phrased as multi-word action phrases (never bare nouns) so this
  never false-trips on legitimate blast-operation-coordination domain
  vocabulary."
  [proposal]
  (let [text (scope-excluded-text proposal)]
    (boolean (some #(str/includes? text %) scope-exclusion-phrases))))

(defn- hard-violations [{:keys [request proposal]} site-record b]
  (let [{:keys [op site-id blaster-id]} proposal]
    (cond-> []
      (nil? site-record)
      (conj {:rule :no-site :detail "未登録 site/blast-operation record"})

      (not= :propose (:effect proposal))
      (conj {:rule :no-actuation :detail "effect は :propose のみ許可（governor は blast-operation 判断を直接実行しない）"})

      (not (contains? allowed-ops op))
      (conj {:rule :unknown-op :detail "closed op-allowlist 外の op（blast-authorization 決定の確定・blast-execution 決定の確定・blasting supervisor/site safety officer の判断の上書きにあたる op は許可されていない）"})

      (and site-id (not= site-id (:site-id request)))
      (conj {:rule :site-mismatch :detail "proposal の site が request で検証済みの site と一致しない"})

      (and blaster-id (nil? b))
      (conj {:rule :unknown-blaster :detail "未登録 blaster への提案は不可"})

      (and b (not= (:site-id b) (:site-id request)))
      (conj {:rule :blaster-wrong-site :detail "blaster が別 site 所属"})

      (scope-exclusion-violation? proposal)
      (conj {:rule :scope-exclusion-violation
             :detail "blast-authorization 決定の確定・blast-execution 決定の確定・blasting supervisor/site safety officer の判断の上書きにあたる提案は恒久的に禁止（human 承認でも上書き不可）"}))))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `blastcoord.store/Store`. Pure — never mutates
  the store, never dispatches a robot action, never handles
  explosives."
  [request _context proposal store]
  (let [site-record (store/site store (:site-id request))
        b (some->> (:blaster-id proposal) (store/blaster store))
        hard (hard-violations {:request request :proposal proposal} site-record b)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        cost (:cost proposal)
        over-threshold? (and (= :coordinate-supply-order (:op proposal))
                              (number? cost) (> cost supply-order-cost-threshold))
        always-risky? (or (contains? always-escalate-ops (:op proposal)) over-threshold?)]
    {:ok? (and (not hard?) (not low?) (not always-risky?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? always-risky?))}))
