**Domain:** Property Maintenance & Repairs (Residential Letting)
**Project:** THORNFIELD-MAINT-001
**Status:** Complete
**Created:** 2026-09-16

---

## 1. Domain Overview — Questions

1. What is the full lifecycle of a maintenance request, from a tenant first noticing an issue to the record being closed?
2. Who are all the distinct actor types who touch a maintenance request, and what can each one see and do?
3. Is a maintenance request ever created by anyone other than the tenant (e.g. a property manager on a routine inspection)?

## 2. Terminology Clarification — Questions

1. What does Thornfield actually call the thing a tenant submits — "ticket," "request," "job," "repair"? Is this consistent across the business, or does the client and the trade side use different words for the same thing?
2. Is there a distinct term for the record once a contractor is actively working it, versus before assignment?

## 3. Core Entities & Lifecycle — Questions

1. What are the distinct states a maintenance request passes through? Is there a difference between "assigned" and "accepted by contractor"?
2. Can a request be reopened after being marked resolved, and if so, under what conditions?
3. Does a single request ever need more than one contractor (e.g. a leak causing both plumbing and drywall damage)?

## 4. Generalization Filter — Questions

1. Which of Thornfield's current triage habits are genuinely standard practice across UK residential letting agents, and which are just how Thornfield happens to do it today?
2. Is the four-severity urgency model (Emergency / Urgent / Routine / Planned) something every letting agent in this domain uses, or is that a Thornfield-specific choice this project would be introducing?

## 5. Regulatory Baseline — Questions (the highest-priority section for this domain)

1. Under the Homes (Fitness for Human Habitation) Act 2018, which specific categories of disrepair carry a statutory expectation of urgent action, and is there an actual defined time window in the Act itself, or does the Act only require action "within a reasonable time" with the reasonableness judged case by case?
2. Does the Housing Health and Safety Rating System (HHSRS) — the hazard-scoring framework local authorities use — factor into how Thornfield should classify severity, or is that a separate regime that doesn't apply to Thornfield directly as a letting agent (as opposed to a local authority enforcement action)?
3. Does the applicable response-time expectation differ between a private-sector tenancy (Thornfield's actual book) and social housing — or is the Fitness for Human Habitation Act's standard the same regardless of landlord type?
4. What does the Housing Ombudsman's Complaint Handling Code actually require in terms of record-keeping and response-time evidence, distinct from the Fitness for Human Habitation Act itself?
5. Is there a single, universal "X hours for emergency, Y days for urgent" table this project can hard-code, or does the honest answer vary by hazard type, severity, and possibly by individual landlord instruction?

## 6. Known Variations — Questions

1. Do different landlords Thornfield manages on behalf of set their own, stricter response-time expectations on top of any statutory minimum?
2. Does the urgency classification differ for a shared/communal-area issue (e.g. a broken external light in a shared stairwell) versus an in-unit issue?

## 7. Typical Scope Boundaries — Questions

1. Is proactive/planned maintenance (e.g. an annual boiler service) in scope for this project, or only reactive tenant-reported issues?
2. Is contractor invoicing/payment in scope, or does that stay in Thornfield's existing finance process?
