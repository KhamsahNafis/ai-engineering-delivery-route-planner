# Evaluation Criteria — Fare-Aware Delivery Route Planner

**Course:** SAIA3113 AI Engineering · Topic 2 Lab, Part B
**Student:** Mohamad Khamsah bin Mohamad Nafis (A24AI0047)
**Version:** 1.0 (Week 2)

Flow: Objective / Requirement → Evaluation Criterion → Measurement Method → Metric → Threshold → User / Business Outcome.
Requirement IDs refer to `docs/week2_requirements.md`.

**Capability buckets for a non-LLM project.** The four buckets were introduced with language-model examples. This project uses AI search and optimisation, so each bucket is interpreted as follows:

- **Domain-specific capability:** quality of the delivery plan (its cost).
- **Instruction-following capability:** the plan obeys the given constraints (deadline, stop limit, capacity) and the system's scope (recommend only, never book).
- **Generation capability:** the plan output is complete and usable by the administrator.
- **Cost and latency:** run time and external-service usage.

## Evaluation Criteria Table

| Objective / Requirement ID | Evaluation Criterion | Capability Bucket | Measurement Method | Metric | Threshold | User / Business Outcome |
|---|---|---|---|---|---|---|
| O3 / NFR2 | Plan cost quality | Domain-specific capability | Run each method on 30 synthetic instances of up to 10 stops and compare its total estimated fare with the exact optimum computed by exhaustive search. | Optimality gap (%) | Mean ≤ 5%; worst case ≤ 10% | The company pays close to the lowest fare achievable for each day's deliveries. |
| O3 / FR3 | Cost compared with current practice | Domain-specific capability | Plan the same 20 synthetic delivery days with the optimiser and with a general-purpose chatbot (fixed prompt), and price both plans with the same fare function. | Share of days where the optimiser's plan costs no more than the chatbot's; mean saving (%) | ≥ 90% of days; mean saving reported (target set after the baseline is measured) | Replacing chatbot planning does not increase delivery cost and shows any saving in ringgit. |
| O1 / FR2 / FR4 / NFR3 | Constraint compliance | Instruction-following capability | Run the automatic constraint checker on every plan from every method (6–30 stops), including instances designed to make the deadline tight. | Share of plans that satisfy stop limit, capacity and deadline, or are explicitly flagged as infeasible | 100% | No booking exceeds the courier's limits and no late delivery is hidden from the administrator. |
| O1 / FR5 / RAI2 | Scope: recommend, never book | Instruction-following capability | Scenario tests through the whole workflow plus a code review for any booking or payment call. | Bookings or payments made by the system | 0 | The business keeps full control of spending; no unintended courier charges. |
| O1 / FR3 / RAI3 | Plan output completeness | Generation capability | Check each plan output against a five-item checklist: booking list, stop order, fare breakdown, arrival times with warnings, estimate/assumption label. | Checklist items present per plan | 5 / 5 on 100% of plans | The administrator can enter the plan into the courier app and explain its cost without extra work. |
| O3 / NFR1 | Planning time | Cost and latency | Time the heuristic and genetic-algorithm methods on 30 synthetic 12-stop instances in Google Colab (CPU). | p90 planning time | ≤ 10 s | The plan is ready within the few minutes available each morning before booking. |
| O3 / NFR4 | Consistency | Cost and latency (operational reliability) | Run each method 10 times on the same 20 instances with a fixed random seed. | Identical-output rate | 100% | The same order list always gives the same plan, so staff can trust and reproduce it. |

The 90% threshold for comparison with chatbot plans is an initial target. It will be reviewed once the baseline cost of the current practice has been measured.

---

## Reflection

**Which evaluation criterion is most important for determining whether the application will be useful to its intended users? Why?**

*Cost compared with current practice* is the most important. The administrator already has a working, if unverified, way of planning: asking a chatbot. The planner is only worth adopting if its plans cost the company no more than that, and ideally less. A small optimality gap against the exact optimum (plan cost quality) is a strong technical result, but on its own it does not show that daily delivery spend changes. *Constraint compliance* is a close second, because one plan that hides a late delivery or breaks the courier's stop limit would cost more trust than many small savings earn. These two together answer the business question: does this tool make delivery planning cheaper without making it riskier?
