---
title: "Sheet Herder - Devlog 8"
date: 2026-04-03
description: "Wrapping up the semester, preparing for the exam and reflecting on what Sheet Herder became."
categories: ["Devlog"]
tags: ["Sheet Herder", "Reflection", "Exam"]
series: ["Sheet Herder"]
series_order: 9
draft: false
---

## Where We Left Off

The application is live, the pipeline is working, and the semester is drawing to a close. This final devlog is less about what was built and more about how it came together — and what I would do differently.

## What Sheet Herder Became

Sheet Herder started as an ambitious idea for a rules-agnostic TTRPG companion app. What it became is a well-structured character sheet manager with a solid backend, a functioning security layer, a tested API, and an automated deployment pipeline. The gap between vision and implementation is real, but it is not a failure — it is scope management.

The domain is clean. The architecture is layered and consistent. The decisions made along the way are documented and defensible. For a first solo backend project of this scale, that feels like the right outcome.

## Technical Decisions in Hindsight

A few decisions stand out on reflection. Keeping associations unidirectional by default added discipline to the entity design and kept the dependency graph manageable. Implementing content hashing for reference data sync was more work than a simple delete-and-reinsert approach, but it is the kind of solution that holds up under scrutiny. Writing tests derived from acceptance criteria rather than chasing coverage numbers meant that every test has a reason to exist.

The JWT fail-fast pattern is a small detail that I am disproportionately satisfied with. A missing environment variable should never surface as a cryptic runtime error. Crashing loudly at startup is the right behaviour.

## What Comes Next

Campaign management, character portrait uploads, and a frontend are the natural next steps. Whether they become part of this project or a separate one is an open question. The backend is built to support them — the domain model accounts for campaigns and membership, and the architecture is clean enough that extending it should not require significant rework.

For now, the focus is the exam. The goal is to walk through the codebase with confidence — not because every decision was perfect, but because every decision was made deliberately.

{{< article link="/posts/devlog-7/" >}}
