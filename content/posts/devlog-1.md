---
title: "Sheet Herder - Devlog 1"
date: 2026-02-13
description: "Domain modelling, business rules, user stories and first steps with JPA."
categories: ["Devlog"]
tags: ["Sheet Herder", "JPA", "Hibernate", "Domain Model"]
series: ["Sheet Herder"]
series_order: 2
showSummary: true
draft: false
---

## Where We Left Off

Last week I committed to developing Sheet Herder — a TTRPG companion app. Since then, a lot of the work has been analytical rather than technical. Before writing a single line of persistence code, I wanted to be confident that the domain was well understood. That turned out to be the right call.

## Domain Model

The domain model for this project is intentionally simple. Do not be fooled — the ERD and class diagrams will be significantly more complex once API integration comes into the picture. For now, the model reflects the core of what the application needs to support: users, characters, campaigns, and the relationships between them.

The most interesting design challenge here was roles. A user is not simply a player or a game master — they can be both, depending on the campaign. This context-dependent role assignment required a shift in how I modelled the domain, and led to the introduction of `CampaignMembership` as an explicit entity rather than a simple join.

## Business Rules

Deciding on business rules had more impact on the design than I initially expected. The rules define the boundaries of the system — what is allowed, what is not, and who is responsible for what. Getting them right early made later implementation decisions easier to reason about.

Most of the rules are defined broadly enough to support other rulesets in a later stage, which was important to me. Sheet Herder is designed around D&D, but the ambition is for it to remain a rules-agnostic companion at its core.

## User Stories

The user stories are intentionally broad. As a solo contributor, fully decomposed tasks would add overhead without adding value. The stories are written to define scope and serve as the basis for acceptance criteria — which in turn will inform how tests are written later in the project.

## Definition of Done

Short but deliberate. The DoD exists as an internal agreement — a checklist that keeps me accountable and prevents scope creep from creeping in through the back door. Every item on it is there for a reason.

## Java Persistence API

This week introduced JPA with a focus on ORM, Hibernate, JPQL and DAOs. Coming from a solid understanding of JDBC and raw SQL, the abstraction layer took some getting used to. The mental model shift from writing queries to mapping objects is significant, and I spent time making sure I understood what Hibernate was doing under the hood rather than treating it as a black box.

The initial implementation reflects the domain model as it stands now — lean entities, no relations yet. That changes next week.

## Current State

I would have liked to be further along, but I would not have spent my time differently. Working through the JPA material carefully — including implementations I may not need for this specific project — gave me a solid foundation to build on.

Sheet Herder is in a good state. The domain is defined, the rules are documented, and the persistence layer is taking shape. The next step is JPA relations, which will be the first real test of how well the domain model holds up in practice.

{{< article path="devlog-0.md" >}}
{{< article path="devlog-2.md" >}}
