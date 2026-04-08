---
title: "Sheet Herder - Devlog 2"
date: 2026-02-20
description: "JPA relations, fetch types and the first real test of the domain model."
categories: ["Devlog"]
tags: ["Sheet Herder", "JPA", "Hibernate", "Relations"]
series: ["Sheet Herder"]
series_order: 3
draft: false
---

## Where We Left Off

Last week I established the domain model and got the basic persistence layer in place. This week the focus shifted to JPA relations — and with it, the first real test of whether the domain model holds up in practice.

## JPA Relations

This week covered all relation types: one-to-one, one-to-many, many-to-one, and many-to-many. Each comes with its own set of trade-offs, and understanding when to use which required more thought than I expected.

The most important decision was directionality. My default approach was to keep associations unidirectional wherever possible — a deliberate choice to reduce coupling between entities. If an entity does not need to know about another, it should not. This kept the model cleaner and the dependency graph easier to reason about.

Fetch types added another layer of consideration. `FetchType.LAZY` is the safer default — it avoids loading data that is not needed. In practice however, reference data entities ended up using `FetchType.EAGER` across the board. Given that this data is always needed when mapping a character sheet, lazy loading would only add complexity without meaningful benefit.

## Cascade and Ownership

Getting cascade behaviour right was one of the more time-consuming parts of this week. The relationship between `User` and `CharacterSheet` is a good example — deleting a user should cascade to all their character sheets. Getting this to behave correctly in Hibernate required careful attention to ownership and orphan removal.

## Current State

The entity layer is taking shape. Relations are in place, fetch types are deliberate, and the cascade behaviour reflects the business rules defined last week. The next step is data integration — fetching reference data from an external API and persisting it locally.

{{< article link="/posts/devlog-1/" >}}
{{< article link="/posts/devlog-3/" >}}
