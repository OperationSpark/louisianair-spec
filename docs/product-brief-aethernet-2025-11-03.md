# Product Brief: Louisianair

- **Date:** 2025-11-03
- **Author:** [Operation Spark](https://www.operationspark.org/) / [shift7](https://shift7.com/)
- **Context:** Open source community initiative focused on accessible air-quality monitoring

---

## Executive Summary

Louisianair empowers fenceline communities to assemble low-cost air monitors, stream trusted readings into a shared open database, and interpret local air trends through accessible dashboards while apprentices and youth build real-world IoT and software skills that strengthen environmental justice capacity.

## Initial Vision

A collaborative OSS program led by Operation Spark and shift7 that equips [fenceline New Orleans communities](https://en.wikipedia.org/wiki/Fenceline_community) with DIY methane and hydrogen sulfide sensors, shared data infrastructure, and approachable dashboards so residents, students, and partner organizations can understand and act on local air insights.

---

## Core Vision

### Problem Statement

Reliable air-quality monitoring tools remain out of reach for most communities because certified sensor kits are expensive, require specialist calibration, and demand technical literacy that neighborhood organizers, schools, and citizen scientists often lack; even when data is collected, legal restrictions and opaque vendor platforms leave residents uncertain how to interpret results or hold polluters accountable.

### Problem Impact

Fenceline neighborhoods lack continuous, trustworthy insight into local air quality because professional-grade monitoring systems remain cost-prohibitive, while schools miss opportunities to turn pollution into hands-on STEM learning. Louisiana’s Community Air Monitoring Reliability Act restricts enforcement to EPA-certified methods that can cost up to $500,000, yet communities still need affordable tools to monitor air quality, educate residents, and organize around local conditions.

### Why Existing Solutions Fall Short

Most commercial-grade monitoring systems bundle proprietary hardware, calibration tooling, and closed dashboards at price points community groups cannot afford, while DIY efforts scatter across incompatible platforms with limited data sharing, leaving no common baseline that advocacy partners, such as [LEAN](https://leanweb.org/about-lean/projects/community-air-monitoring-network), can extend or rely on.

### Proposed Solution

Build an OSS toolkit that combines guidance on assembling off-the-shelf particulate and gas sensors with Arduino/Raspberry Pi controllers, provides firmware and dashboards for live readings, and publishes an open data service so communities and researchers can access and contribute to a shared air-quality dataset, complete with plain-language visualizations that help residents understand what the readings mean for their daily lives.

### Key Differentiators

- Central aggregation layer that pools crowd-sourced sensor data, keeps it openly accessible via API endpoints, and surfaces ready-to-use dashboards.
- Educational playbooks that translate sensor assembly, calibration, and data literacy into approachable guides for students, apprentices, and community organizers.
- Intentional environmental justice alignment that pairs hardware deployments with youth workforce development and community partnerships.

---

## Target Users

### Primary Users

Community-based organizations, K‑12 and university programs, and grassroots research collaboratives that want to capture trustworthy local air-quality readings without investing in proprietary environmental monitoring infrastructure.

---

## Success Metrics

Louisianair’s pilot is considered successful when it simultaneously empowers community partners with actionable data, produces reusable open tooling, and delivers measurable workforce development outcomes.

### Business Objectives

- Strengthen environmental justice capacity in fenceline New Orleans neighborhoods by co-delivering open hardware, software, and data literacy resources.
- Provide meaningful workforce development for Operation Spark apprentices and students through real-world IoT and open data projects.

### Key Performance Indicators

- Deploy 5–10 methane and hydrogen sulfide sensor units built with Raspberry Pi controllers across target communities.
- Maintain a shared data pool and public API that external partners can query for real-time and historical readings.
- Publish a repeatable playbook and starter project that documents how to assemble kits, stand up dashboards, and contribute new nodes.
- Engage 15–25 students and apprentices across the program phases in hands-on technical roles.

---

## MVP Scope

### Core Features

- DIY sensor kit blueprint leveraging Raspberry Pi and industrial-grade air-quality sensors tuned for methane and hydrogen sulfide detection.
- Cloud-backed data ingestion pipeline that normalizes readings from 5–10 pilot nodes and stores them in a shared repository.
- Public API that exposes real-time and historical readings for reuse in school projects, research, and community dashboards.
- Web dashboard with accessible explanations, visualizations, and alerting concepts tailored for non-technical audiences.
- Open playbook and starter project that document hardware assembly, deployment steps, and sample code so partners can replicate or extend the network.

### Out of Scope (Pilot)

- Pursuing EPA-certified instrumentation or accredited lab analysis during the initial deployment.
- Manufacturing or distributing turnkey hardware kits beyond the pilot cohort.
- Building compliance-grade enforcement tooling or legal advocacy workflows within the MVP dashboard.

### MVP Success Criteria

- 5–10 sensor units deployed in fenceline New Orleans communities with reliable methane and hydrogen sulfide readings flowing into the shared database.
- Open API promoted for community, research, and industry consumption with documented query examples.
- Playbook and starter projects (Node.js/Python) published to guide external developers in building kits, dashboards, and data-consuming applications.
- Workforce outcomes achieved by engaging at least three apprentices and 10–30 youth in hands-on IoT, software, and data practices throughout the pilot.

### Future Vision Features

- Expand the sensor network beyond New Orleans through partnerships such as LEAN’s Community Air Monitoring Network and other EJ coalitions.
- Layer advanced analytics, alerting, and storytelling dashboards tailored for policymakers and healthcare advocates.
- Host seasonal hackathons and workshops (Summer/Fall 2026) that extend tooling, produce community apps, and onboard new contributors.
- Explore pathways toward certified data collection in collaboration with regulators without losing accessibility.

---

## Risks and Assumptions

- Community-friendly device onboarding is still being evaluated; the pilot must test whether manual uploads, scheduled API pushes, or streaming connectors offer the right balance of reliability and accessibility for partner-run monitors.
- Continued collaboration among Operation Spark, shift7, and LEAN is assumed to align sensor placement, data governance, and storytelling needs.
- Sustained funding and staffing are required to keep apprentices and youth engaged across the full pilot timeline and follow-on workshops.

---

## Timeline

- **Oct–Dec 2025:** Planning, sensor design, component procurement, and early prototyping; kick off coordination between Operation Spark, shift7, and partner leads (Harvey, Leanna).
- **Jan 2026:** IoT class and apprentices begin structured work; instructors staff prototyping and preparation.
- **Feb–Mar 2026:** IoT students build and deploy 5–10 sensor units; database design and integration work accelerates.
- **Mar–Apr 2026:** Apprentices finish the API, connect deployed sensors to the shared data pool, and validate outputs.
- **Apr 2026:** Full deployment, debugging, and operational readiness checks.
- **Apr/May 2026:** Publish the open API, playbook, and starter projects for external developers.
- **Summer–Fall 2026 (funding dependent):** Potential hackathons or workshops to extend the platform, onboard partners, and explore new deployments.

---

## Technical Preferences

- API layer: Prefer Go for its strong typing, straightforward testing, low resource footprint, and maintainability, while remaining open to Node.js if onboarding considerations for entry-to-mid developers outweigh the benefits.
- Edge bridge software: Raspberry Pi services likely written in JavaScript or Python to interface with sensor hardware and normalize payloads for upstream ingestion.
- Frontend: Need a framework with solid DX and deployability without leaning on the React/Vercel combo; exploring modern alternatives (e.g., SvelteKit, Astro, or Next.js with server-friendly hosting) that support accessible visualization and admin tooling.
- Hosting: Currently satisfied with Fly.io, exploring AWS to broaden operational knowledge, and keeping existing GCP projects in play; whichever path offers the best mix of low cost, observability, deployment simplicity, and long-term maintainability will lead.
- Database: Undecided; willing to consider managed relational or time-series stores that balance ease of use with open data access patterns.

---

## Organizational Context

- Core delivery partnership between Operation Spark and shift7, combining environmental justice experience with youth workforce development.
- Targeted collaboration with LEAN’s Community Air Monitoring Network to augment existing deployments and align data-sharing practices.
- Anticipate coordination with local fenceline neighborhoods, schools, and research allies to prioritize sensor placement and program impact.

---

_This Product Brief captures the vision and requirements for louisianair._

_It was created through collaborative discovery and reflects the unique needs of this TBD project._

_Next: Use the PRD workflow to create detailed product requirements from this brief._

<!-- Final refinements: incorporated fenceline terminology, documented onboarding risk, and refreshed executive summary for final handoff. -->
