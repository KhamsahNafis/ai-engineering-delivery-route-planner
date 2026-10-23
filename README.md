# Fare-Aware Delivery Route Planner for SME Courier Bookings

> Plans the cheapest way to split a day's deliveries into on-demand courier bookings and order the stops, for the delivery administrator of a Malaysian SME food manufacturer.

**Course:** SAIA3113 AI Engineering, Universiti Teknologi Malaysia
**Student:** Mohamad Khamsah bin Mohamad Nafis (A24AI0047)

## Problem Statement

A Malaysian SME food manufacturer delivers to 6 to 12 business customers a day through an on-demand courier, starting from its factory with a 6:00 pm deadline. The administrator currently asks a general-purpose chatbot to group and order the stops, but the chatbot does not compute distances or courier fares, so plans carry no cost guarantee and can change for the same input. Choosing the cheapest grouping and order is a combinatorial search problem, which AI search and optimisation methods can solve systematically with a verifiable cost.

## Evidence of the Problem

- Last-mile delivery accounts for 41% of total supply chain costs (Capgemini Research Institute, 2019, [*The last-mile delivery challenge*](https://www.capgemini.com/wp-content/uploads/2019/01/Report-Digital-%E2%80%93-Last-Mile-Delivery-Challenge1.pdf)).
- The courier's fare depends on bookings and stops, not only distance: a car in the Klang Valley is charged RM 3.00 per additional stop on top of a per-booking delivery fee ([Lalamove Malaysia pricing](https://www.lalamove.com/en-my/all-vehicle-pricing-detail)).
- LLMs reach optimal routes only on small problems; their optimality gap grows quickly with size and simple heuristics outperform them on larger instances (Yang et al., 2023, [*Large Language Models as Optimizers*](https://arxiv.org/abs/2309.03409)).

## Initial Related Work / Existing Approaches

- **Lalamove "Optimize Route"** ([blog](https://www.lalamove.eu/en-de/blog/save-time-and-costs-with-lalamoves-multi-stop-feature), [API](https://developers.lalamove.com/)): reorders stops inside one booking only; does not split stops into bookings, and is not available in the mobile app used by the company.
- **Google OR-Tools** ([docs](https://developers.google.com/optimization/routing)): general routing solver that minimises distance or time for an owned fleet; courier fares must be modelled by the user.
- **VROOM / openrouteservice** ([VROOM](https://github.com/VROOM-Project/vroom)): fast routing heuristics for owned vehicles; no courier fare or booking-split decision.
- **LLMs as optimisers** ([Yang et al., 2023](https://arxiv.org/abs/2309.03409)): flexible, but solution quality degrades with problem size and varies between runs.

## Engineering Gap / Limitation

No existing approach decides how to split a day's deliveries into courier bookings **and** order each booking to minimise the total courier fare from a fixed depot under a same-day deadline. Distance-based solvers assume an owned fleet, the courier's own feature works within a single booking on the web only, and chatbot planning gives no cost guarantee.

## Project Objectives

- O1: To design a fare-aware route optimisation model that splits a day's deliveries into courier bookings and orders the stops from a fixed depot, subject to a delivery deadline and the courier's stop limit.
- O2: To develop the optimiser using AI search methods (exact search for small instances, nearest neighbour with 2-opt, and a genetic algorithm), tested on synthetic instances built from public map data.
- O3: To evaluate the methods on total estimated fare, deadline feasibility, computation time and consistency, compared with chatbot-produced plans.

## Target Users

Delivery administrators of Malaysian SMEs that outsource same-day deliveries to on-demand couriers, planning 6 to 12 deliveries a day, often on a smartphone.

## Data Sources

- Lalamove Malaysia vehicle pricing page — https://www.lalamove.com/en-my/all-vehicle-pricing-detail
- Lalamove Delivery API v3 documentation — https://developers.lalamove.com/
- OpenStreetMap extract for Malaysia (Geofabrik, ODbL) — https://download.geofabrik.de/asia/malaysia-singapore-brunei.html
- openrouteservice time-distance matrix service — https://openrouteservice.org/services/

Only synthetic delivery instances are stored in this repository. No real customer names, addresses, contact numbers or receipts are committed.

## Repository Structure

```text
docs/         project documentation (Week 1 project scope)
data/         synthetic delivery instances (no real customer data)
src/          optimiser source code
notebooks/    experiments
evaluation/   evaluation scripts and results
prompts/      kept for the course structure; this project does not use an LLM
```

## Weekly Progress

| Week | Focus | Deliverable | Status |
|------|-------|-------------|--------|
| W1 | Project scope + repository setup | README + docs/week1_project_scope.md | ✅ |
