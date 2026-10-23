# Week 1 Project Scope

**Course:** SAIA3113 AI Engineering
**Student:** Mohamad Khamsah bin Mohamad Nafis (A24AI0047)
**Status:** Preliminary scope (Week 1). To be refined in Topic 2.

---

## Project Name

Fare-Aware Delivery Route Planner for SME Courier Bookings

## Domain

Logistics / Last-mile delivery planning (route optimisation for outsourced on-demand courier deliveries)

## Problem Statement

A Malaysian SME food manufacturer delivers its products to 6 to 12 business customers (mostly petrol stations) every day through an on-demand courier (Lalamove). All deliveries start from the factory, are one-way, use a car, and must arrive before 6:00 pm. An administrator plans them by hand: the day's delivery list is pasted into a general-purpose AI chatbot, which is asked to group the stops into separate courier bookings and to order the stops in each group, and the result is then typed stop by stop into the courier's mobile app.

This planning has no cost guarantee. The chatbot does not compute road distances or courier fares, and it can return a different plan for the same list. The courier's fare depends on how stops are grouped into bookings (each booking has its own delivery fee plus a charge per extra stop), not only on distance, so a short route is not always the cheapest plan.

AI is appropriate because choosing the cheapest grouping and order of stops is a combinatorial search problem: even 12 stops can be split and ordered in millions of ways, which is beyond manual or rule-based planning. Classical AI search and optimisation methods (exact search, heuristics and metaheuristics) can explore this space systematically and give a cost that can be checked.

## Evidence of the Problem

**Evidence 1:** Last-mile delivery is the most expensive part of the delivery chain, accounting for 41% of total supply chain costs, so poor last-mile planning has a direct cost impact.
Source / citation: Capgemini Research Institute (2019), *The last-mile delivery challenge*. https://www.capgemini.com/wp-content/uploads/2019/01/Report-Digital-%E2%80%93-Last-Mile-Delivery-Challenge1.pdf

**Evidence 2:** The courier's public tariff charges per booking and per stop: for a car in the Klang Valley, each additional stop costs RM 3.00, while the base delivery fee is shown only in the courier app. The cheapest plan therefore depends on how stops are grouped into bookings, which distance-only planning does not capture.
Source / citation: Lalamove Malaysia, *Pricing / All vehicle pricing detail*. https://www.lalamove.com/en-my/all-vehicle-pricing-detail

**Evidence 3:** Large language models are unreliable as route optimisers. In the OPRO study, LLMs found optimal routes for 10-city travelling salesman problems, but their optimality gaps grew quickly with problem size and a simple farthest-insertion heuristic outperformed all tested LLMs on larger instances.
Source / citation: Yang, C. et al. (2023). *Large Language Models as Optimizers*. arXiv:2309.03409. https://arxiv.org/abs/2309.03409

**Evidence 4 (practice):** At the author's workplace, delivery plans are currently produced with a general-purpose chatbot and re-typed into the courier app every working day; plans and fares are not recorded, so their cost cannot be checked afterwards.

## Initial Related Work (at least 3)

**Study / solution 1:** Lalamove multi-stop "Optimize Route" feature
Source / citation: Lalamove (EU), *Save time and costs with Lalamove's multi-stop feature*, https://www.lalamove.eu/en-de/blog/save-time-and-costs-with-lalamoves-multi-stop-feature ; Lalamove Delivery API v3 (`isRouteOptimized`), https://developers.lalamove.com/
Key limitation: It only reorders the stops inside one booking that the user has already composed (up to 20 stops). It does not decide how to split a day's stops into several bookings, and in our use it is available on the web app but not in the mobile app the company uses.

**Study / solution 2:** Google OR-Tools vehicle routing solver
Source / citation: Google, *OR-Tools: Vehicle Routing*. https://developers.google.com/optimization/routing
Key limitation: A general-purpose solver library. Its examples minimise distance or time for a fleet the user owns; a courier's per-booking and per-stop fare must be modelled by the user, and it is not connected to any booking workflow.

**Study / solution 3:** VROOM optimisation engine (also offered through the openrouteservice API)
Source / citation: VROOM Project, https://github.com/VROOM-Project/vroom ; openrouteservice, https://openrouteservice.org/services/
Key limitation: Fast heuristics for routing a company's own vehicles by travel time or distance; it has no notion of a third-party courier fare or of choosing how many bookings to make.

**Study / solution 4:** LLMs used directly as optimisers (the current practice)
Source / citation: Yang, C. et al. (2023), *Large Language Models as Optimizers*, arXiv:2309.03409.
Key limitation: Solution quality degrades as the number of stops grows, results vary between runs, and the model does not compute real distances or fares.

**Study / solution 5:** Vehicle routing with partial outsourcing to a common carrier
Source / citation: *The Vehicle Routing Problem with Partial Outsourcing*, Transportation Science 54(4), 2020, pp. 1034–1052.
Key limitation: Assumes the company owns a fleet and outsources only some customers; it does not model a business that outsources every delivery to an on-demand courier.

## Engineering Gap / Limitation

Existing tools either optimise distance for a company's own fleet (OR-Tools, VROOM, commercial route planners), reorder stops only inside a single courier booking (Lalamove's own feature, web only), or rely on a chatbot that does not compute distances or fares. None of them decides **how to split a day's deliveries into courier bookings and order each booking to minimise the total courier fare**, starting from a fixed depot with a same-day deadline, in a form a small business can use from a phone.

## Initial Project Objectives (2–3)

- **O1:** To design a fare-aware route optimisation model that splits a day's deliveries into courier bookings and orders the stops from a fixed depot, subject to a delivery deadline and the courier's stop limit.
- **O2:** To develop the optimiser using AI search methods (exact search for small instances, nearest-neighbour construction with 2-opt improvement, and a genetic algorithm), tested on synthetic delivery instances built from public map data.
- **O3:** To evaluate the methods on total estimated fare, deadline feasibility, computation time and consistency, compared with plans produced by a general-purpose chatbot.

## Target Users

Delivery administrators of Malaysian SMEs that outsource same-day deliveries to on-demand couriers; the primary user is the administrator of a Malaysian SME food manufacturer who plans 6 to 12 deliveries per day on a smartphone. The owner or manager is a secondary user who reviews delivery cost.

## Data Sources (at least 2)

- **Source 1:** Lalamove Malaysia vehicle pricing page (tariff structure, additional stop fee) — URL: https://www.lalamove.com/en-my/all-vehicle-pricing-detail
- **Source 2:** Lalamove Delivery API v3 documentation (quotation price breakdown, stop limits, route optimisation flag) — URL: https://developers.lalamove.com/
- **Source 3:** OpenStreetMap data extract for Malaysia (public locations such as petrol stations, used to build synthetic delivery instances; ODbL licence) — URL: https://download.geofabrik.de/asia/malaysia-singapore-brunei.html
- **Source 4 (tool documentation):** openrouteservice time-distance matrix service — URL: https://openrouteservice.org/services/

No company customer list, addresses, receipts or other confidential data will be committed to this repository. Real delivery data stays with the company; the public repository uses synthetic instances only.

## Similar to EduAdvisor

- Starts from a real, evidenced problem for a clearly defined user group.
- Uses publicly available documents and data as its sources.
- Is a decision-support tool: a person reviews the output before acting on it.
- Is evaluated objectively against a baseline, and its Responsible AI risks are identified from the start.

## Different from EduAdvisor

- EduAdvisor is a document-grounded question-answering assistant (LLM with retrieval); this project uses classical AI search and optimisation, with no LLM.
- EduAdvisor's input is natural-language questions over PDFs; this project's input is a list of delivery locations, a road distance matrix and a fare model.
- EduAdvisor outputs text answers; this project outputs a delivery plan (bookings and stop order) with an estimated cost.
- EduAdvisor is evaluated on groundedness and answer quality; this project is evaluated on cost, feasibility (deadline and stop limit), optimality gap, run time and consistency.

## Responsible AI Risks (at least 2)

- **Risk 1 — Incorrect or infeasible plans:** wrong distance or fare estimates could produce a plan that is more expensive than expected or misses the 6:00 pm deadline, harming the business and its customers. Mitigation: deadline and stop-limit checks on every plan, showing the cost breakdown, and requiring the administrator to confirm before booking.
- **Risk 2 — Privacy of customer data:** delivery addresses and contact numbers are personal or business information. Mitigation: synthetic data in the public repository, anonymisation in reports, and no real customer data committed (in line with the Personal Data Protection Act 2010).
- **Risk 3 — Unequal service and over-reliance:** a cost-only optimiser may always place remote customers last, so they routinely receive goods close to the deadline; staff may also accept plans without checking them (automation bias). Mitigation: report each stop's estimated arrival time, flag stops near the deadline, and keep the human decision explicit.
