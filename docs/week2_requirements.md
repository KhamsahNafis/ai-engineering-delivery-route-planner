# Requirements Specification — Fare-Aware Delivery Route Planner

**Course:** SAIA3113 AI Engineering · Topic 2 Lab, Part A
**Student:** Mohamad Khamsah bin Mohamad Nafis (A24AI0047)
**Version:** 1.0 (Week 2)

This document turns the Week 1 project scope (`docs/week1_project_scope.md`) into a traceable engineering specification:
Project Framing → Application Scope → Requirements → Evaluation Criteria → V1 Architecture.

---

## Section 0 — Project Framing & Application Scope Checkpoint

| Framing element | Content |
|---|---|
| **Problem Evidence** | (1) Last-mile delivery accounts for 41% of total supply chain costs, so planning quality has a direct cost impact (Capgemini Research Institute, 2019, *The last-mile delivery challenge*). (2) The courier's fare depends on bookings and stops, not only distance: a car in the Klang Valley is charged RM 3.00 per additional stop on top of a per-booking delivery fee (Lalamove Malaysia pricing page). (3) LLMs used as route optimisers reach optimal answers only on small problems, and simple heuristics outperform them as the problem grows (Yang et al., 2023, *Large Language Models as Optimizers*, arXiv:2309.03409). (4) Observed practice at a Malaysian SME food manufacturer: plans for 6–12 daily deliveries are produced by a general-purpose chatbot and re-typed into the courier's mobile app; no plan or fare is recorded. |
| **Existing Approaches** | (1) **Lalamove "Optimize Route"**: reorders the stops of one booking (up to 20 stops) on the Lalamove web app. (2) **Google OR-Tools**: general vehicle routing solver that minimises distance or time for a fleet the user owns. (3) **VROOM / openrouteservice**: fast routing heuristics for owned vehicles. (4) **General-purpose chatbots** (current practice): flexible natural-language planning. (5) **VRP with partial outsourcing** (Transportation Science, 2020): models a private fleet that outsources some customers to a carrier. |
| **Project / Engineering Gap** | None of these decides **how to split a day's deliveries into courier bookings and order each booking to minimise the total courier fare**, from a fixed depot under a same-day deadline. The courier's own feature only reorders one booking and is web-only; solvers assume an owned fleet and a distance objective; chatbot plans are unverified. This is a practical engineering gap, not a claim of scientific novelty. |
| **Objective O1** | To **design** a fare-aware route optimisation model that splits a day's deliveries into courier bookings and orders the stops from a fixed depot, subject to a delivery deadline, vehicle capacity and the courier's stop limit. |
| **Objective O2** | To **develop** the optimiser using AI search methods (exact search for small instances, nearest-neighbour construction with 2-opt improvement, and a genetic algorithm), tested on synthetic delivery instances built from public map data. |
| **Objective O3** | To **evaluate** the methods on total estimated fare, constraint feasibility, computation time and consistency, compared with plans produced by a general-purpose chatbot and with the exact optimum. |
| **Intended Users** | Primary: the delivery administrator of a Malaysian SME food manufacturer who plans 6–12 courier deliveries a day, often on a smartphone. Secondary: the owner/manager who reviews delivery cost. |
| **Supported Tasks** | (1) Enter or load the day's delivery list (stops with coordinates and parcel counts). (2) Generate a delivery plan: bookings, stop order, estimated fare and estimated arrival time per stop. (3) Show warnings for stops that are late or close to the deadline. (4) Compare plans from different methods on the same list. (5) Produce a copy-ready, per-booking stop list for manual entry in the courier app. |
| **Information / Knowledge Domain** | Last-mile delivery from one fixed depot in the Klang Valley to business customers in the Klang Valley, Negeri Sembilan and Melaka; road distances and travel times; the on-demand courier's public tariff structure (per-booking fee, distance charge, additional-stop fee) and stop limit. |
| **Operating Context** | Used each working morning, before bookings are made, by one administrator on a phone or laptop. V1 is a prototype (Colab notebook with a simple planner view) on synthetic instances; the administrator reviews the plan and books manually in the courier app. |
| **Important Constraints** | (1) **Data:** the public repository may hold synthetic data only; no real customer names, addresses or phone numbers. (2) **Tariff:** the courier's base delivery fee is not published, so V1 uses a configurable tariff function with assumed parameters, to be calibrated later. (3) **Access:** distances come from a free routing service with request limits, or a straight-line fallback. (4) **Time:** one semester; code runs in Google Colab. (5) **Policy:** no paid bookings are placed by the system. |
| **Explicit Out-of-Scope Tasks** | The system shall not place, pay for or cancel courier bookings; decide which orders are delivered on a given day; re-route drivers in real time; plan for a company-owned fleet or for other couriers; forecast demand; or give advice outside delivery planning. |
| **Evaluation Focus** | Does the optimiser produce plans that always satisfy the deadline, capacity and stop-limit constraints, cost no more than the chatbot-made plans for the same deliveries, stay close to the exact optimum, and run fast and consistently enough to use every morning? |

---

## Section 1 — Functional Requirements

| # | Requirement — The system SHALL | Scope Boundary — SHALL NOT (where relevant) |
|---|---|---|
| FR1 | Accept a delivery list of 1 to 30 stops, each with an identifier, coordinates and a parcel count, together with one fixed depot location. | Silently accept a stop with missing or invalid coordinates; it shall be rejected and reported to the user. |
| FR2 | Produce a delivery plan that assigns every stop to exactly one courier booking and gives the stop order of each booking, starting from the depot (one-way, no return). | Exceed the courier's limit of 15 drop-offs per booking or the vehicle's parcel capacity. |
| FR3 | Show the estimated fare of each booking and of the whole plan, with its breakdown (per-booking fee, distance charge, additional-stop fees). | Present an estimated fare as the courier's actual price; every figure shall be labelled as an estimate. |
| FR4 | Show the estimated arrival time at every stop and flag any stop that arrives after 6:00 pm or within 30 minutes of it. | Return a plan that misses the deadline without marking it as infeasible. |
| FR5 | Produce a copy-ready list for each booking (stops in order with their address fields) so the administrator can enter it in the courier app. | Place, pay for or cancel a courier booking; the administrator remains responsible for booking. |
| FR6 | Let the user run more than one planning method on the same delivery list and compare their total estimated fares and run times. | Switch the method used for the user's plan without telling the user. |

---

## Section 2 — Non-Functional Requirements

| # | Quality Attribute | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| NFR1 | Performance | A plan for a typical day shall be produced quickly enough for morning use. | Run the heuristic and genetic-algorithm methods on 30 synthetic 12-stop instances in Google Colab (CPU) and time each run. | p90 planning time | ≤ 10 s for 12 stops |
| NFR2 | Correctness (plan quality) | Fast methods shall produce plans close to the cheapest possible plan under the fare model. | On 30 synthetic instances of up to 10 stops, compare each method's total estimated fare with the exact optimum. | Optimality gap (%) | Mean ≤ 5%; worst case ≤ 10% |
| NFR3 | Reliability | Every plan returned shall satisfy the hard constraints. | Run an automatic constraint checker on every plan produced in the evaluation set (all methods, 6–30 stops). | Share of plans that satisfy stop limit, capacity and deadline (or are explicitly flagged) | 100% |
| NFR4 | Consistency | The same input shall always give the same plan. | Run each method 10 times on the same 20 instances with a fixed random seed and compare outputs. | Identical-output rate | 100% |
| NFR5 | Cost | Planning shall stay within the free usage of the routing service. | Count routing-service requests per plan during evaluation. | Distance-matrix requests per plan | ≤ 1 (matrix cached per instance); RM 0 per plan |
| NFR6 | Maintainability | A change in the courier's tariff shall not require code changes. | Change the tariff parameters (base fee, per-km rate, stop fee) in the configuration file and re-run a plan. | Time to apply a tariff update | ≤ 5 min, configuration file only |

---

## Section 3 — Responsible AI Requirements

| # | Responsible AI Risk | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| RAI1 | Privacy | The repository and any external service calls shall contain no personal or customer-identifying data; stops are identified by codes and coordinates only. | Scan the repository for names, phone numbers and addresses; inspect the payloads sent to the routing service during a test run. | Number of PII items found | 0 |
| RAI2 | Human oversight / autonomy | The system shall only recommend a plan; a person reviews it and makes every booking. | Scenario tests covering plan generation, warnings and export; code review for any booking or payment call. | Bookings or payments made by the system | 0 (no booking function in V1) |
| RAI3 | Transparency | Every plan shall show its fare breakdown, the tariff assumptions used, and that all figures are estimates. | Inspect the output of every plan in the evaluation set against a checklist (breakdown, assumption note, estimate label, method name). | Share of plans with all checklist items | 100% |
| RAI4 | Fairness of service | The planner shall not routinely push the same customers to the end of the day without making this visible. | Over 30 synthetic delivery days with the same customer pool, record how often each customer is scheduled within 30 minutes of the deadline, and check that each such stop carries a warning. | Warning coverage for near-deadline stops; maximum share of days any single customer is near-deadline | 100% warning coverage; any customer above 50% of days is reported for review |
| RAI5 | Reliability / over-reliance | The system shall clearly mark when its estimates may be unreliable (for example, straight-line distances used because the routing service was unavailable). | Disable the routing service in a test run and inspect the output. | Share of affected plans carrying a reliability warning | 100% |

---

## Reflection

**Which Responsible AI requirement will be the most difficult to measure or enforce technically? Why?**

RAI4 (fairness of service) is the hardest. The other requirements can be checked on a single plan: a PII scan either finds data or it does not, and a plan either shows its fare breakdown or it does not. Fairness of service only shows up across many days. Whether a customer is always served last depends on geography (remote stations are naturally at the end of a route) and on which customers order together, so a pattern that looks unfair may simply be the cheapest feasible order. Measuring it needs a realistic multi-day order history, which the project does not have yet, and the 50% review threshold is a starting assumption rather than an agreed policy. Enforcing it also conflicts directly with the cost objective: forcing a remote customer earlier usually makes the plan more expensive. For V1 the requirement is therefore to make the pattern visible through per-stop warnings and a report, and to leave the trade-off decision to the people running the business.
