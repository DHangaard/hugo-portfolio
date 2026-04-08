---
title: "Sheet Herder - Devlog 3"
date: 2026-02-27
description: "Fetching reference data from the D&D 5e API, threading with ExecutorService and mapping with DTOs."
categories: ["Devlog"]
tags: ["Sheet Herder", "API Integration", "ExecutorService", "DTO"]
series: ["Sheet Herder"]
series_order: 4
showSummary: true
draft: false
---

## Where We Left Off

The entity layer is in place and relations are working. This week the focus shifted outward — fetching data from an external API and integrating it into the persistence layer.

## ExecutorService, Futures and Callables

This week reintroduced threading through `ExecutorService`, `Future` and `Callable`. The practical application here was concurrent API fetching — rather than fetching races, languages, traits and subraces sequentially, each fetch runs in its own thread and the results are collected once all are complete.

The difference in startup time is noticeable. More importantly, it introduced a pattern that is used throughout the application: submit tasks, collect futures, handle results.

## DTOs and JSON Mapping

Fetching data from an external API means dealing with JSON responses that do not map cleanly to your own entities. DTOs serve as the translation layer — they model the external response, and the service layer is responsible for converting them into domain entities before anything is persisted.

This separation matters. The domain model should not be shaped by the structure of an external API. Keeping DTOs distinct from entities meant that changes to the API response format would not ripple through the entire codebase.

## D&D 5e API Integration

The D&D 5e REST API provided all the reference data needed for character creation — races, subraces, languages and traits. Fetching this data at startup and persisting it locally was a deliberate choice. It avoids runtime dependency on the external API and gives the application full control over the reference data it works with.

One challenge was handling updates. If the API data changes between deployments, the local data should reflect that without unnecessary writes. This led to the introduction of content hashing — a SHA-256 hash computed from each record's fields, used to detect changes and skip unchanged records on every startup.

## Current State

Reference data is fetched at startup, persisted locally, and kept current through hash-based change detection. The character sheet now has something meaningful to reference. Next up: building the REST API layer on top of what has been built so far.

{{< article link="/posts/devlog-2/" >}}
{{< article link="/posts/devlog-4/" >}}
