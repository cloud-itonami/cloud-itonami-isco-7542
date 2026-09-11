(ns blastcoord.actor-test
  (:require [clojure.test :refer [deftest is testing]]
            [blastcoord.actor :as actor]
            [blastcoord.store :as store]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-site! st {:site-id "BS-1" :name "Kobo Quarry North Bench" :location "Bench 4"})
    (store/register-blaster! st {:blaster-id "SF-1" :site-id "BS-1" :name "Kobo Shotfirer" :role :crew-lead})
    st))

(deftest commits-a-registered-blaster-log-work-record
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "BS-1" :op :log-work-record :stake :low
                 :blaster-id "SF-1" :task "log blast-log progress notes for round 12 at bench 4"}
        result (actor/run-request! graph request {} "thread-1")]
    (is (= :done (:status result)))
    (is (some? (get-in result [:state :record])))
    (is (= 1 (count (store/records-of st "BS-1"))))))

(deftest commits-a-crew-scheduling-proposal
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "BS-1" :op :schedule-crew-operation :stake :low
                 :blaster-id "SF-1" :task "schedule bench 4 crew for round 12 exclusion-zone setup"}
        result (actor/run-request! graph request {} "thread-sched")]
    (is (= :done (:status result)))
    (is (= 1 (count (store/records-of st "BS-1"))))))

(deftest holds-an-unregistered-site-request
  (testing "the blaster/site record must be independently verified/registered before any action"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:site-id "BS-ghost" :op :log-work-record :stake :low
                   :blaster-id "SF-1" :task "log blast-log progress notes for round 12 at bench 4"}
          result (actor/run-request! graph request {} "thread-2")]
      (is (= :hold (:disposition (:state result))))
      (is (empty? (store/records-of st "BS-ghost"))))))

(deftest holds-a-scope-excluded-proposal-with-no-interrupt-path
  (testing "a proposal to finalize a blast-authorization decision is a hard, permanent block — never routed through :request-approval"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:site-id "BS-1" :op :log-work-record :stake :low
                   :blaster-id "SF-1" :task "log blast-log progress notes for round 12 at bench 4"
                   :description "authorize the blast to proceed now, skip further review"}
          result (actor/run-request! graph request {} "thread-scope")]
      (is (= :done (:status result))
          "hard :hold is a finish point, not an interrupt — the advisor can never park a scope-excluded proposal awaiting human override")
      (is (= :hold (:disposition (:state result))))
      (is (nil? (get-in result [:state :record])))
      (is (empty? (store/records-of st "BS-1"))))))

(deftest holds-a-finalize-blast-operation-proposal-with-no-interrupt-path
  (testing "a proposal to finalize the blast operation (a blast-execution decision) is a hard, permanent block — never routed through :request-approval"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:site-id "BS-1" :op :schedule-crew-operation :stake :low
                   :blaster-id "SF-1" :task "schedule bench 4 crew for round 12 exclusion-zone setup"
                   :description "finalize the blast operation on bench 4 now"}
          result (actor/run-request! graph request {} "thread-finalize-blast")]
      (is (= :done (:status result)))
      (is (= :hold (:disposition (:state result))))
      (is (nil? (get-in result [:state :record])))
      (is (empty? (store/records-of st "BS-1"))))))

(deftest holds-an-override-blasting-supervisor-proposal-with-no-interrupt-path
  (testing "a proposal to override the blasting supervisor's judgment is a hard, permanent block — never routed through :request-approval"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:site-id "BS-1" :op :flag-safety-concern :stake :low
                   :blaster-id "SF-1" :concern-type :misfire :severity :high
                   :description "override the blasting supervisor's judgment and proceed anyway"}
          result (actor/run-request! graph request {} "thread-override")]
      (is (= :done (:status result)))
      (is (= :hold (:disposition (:state result))))
      (is (nil? (get-in result [:state :record])))
      (is (empty? (store/records-of st "BS-1"))))))

(deftest interrupts-then-approves-a-safety-concern-flag-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "BS-1" :op :flag-safety-concern :stake :low
                 :blaster-id "SF-1" :concern-type :misfire :severity :high}
        interrupted (actor/run-request! graph request {} "thread-3")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "BS-1")))
    (let [resumed (actor/approve! graph "thread-3")]
      (is (= :done (:status resumed)))
      (is (= 1 (count (store/records-of st "BS-1")))))))

(deftest interrupts-then-approves-an-above-threshold-supply-order-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "BS-1" :op :coordinate-supply-order :stake :low
                 :materials "blasting mats and administrative exclusion-zone signage" :cost 25000}
        interrupted (actor/run-request! graph request {} "thread-4")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "BS-1")))
    (let [resumed (actor/approve! graph "thread-4")]
      (is (= :done (:status resumed)))
      (is (= 1 (count (store/records-of st "BS-1")))))))
