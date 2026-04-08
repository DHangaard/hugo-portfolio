---
title: "Sheet Herder - Devlog 5"
date: 2026-03-13
description: "REST Assured, Hamcrest, Gherkin syntax, Testcontainers and writing tests that mean something."
categories: ["Devlog"]
tags: ["Sheet Herder", "Testing", "REST Assured", "Testcontainers"]
series: ["Sheet Herder"]
series_order: 6
showSummary: true
draft: false
---

## Where We Left Off

The API layer is in place. This week the focus shifted to testing — which in this project means more than just confirming that endpoints return 200.

## REST Assured and Hamcrest

REST Assured makes it possible to write integration tests that read almost like plain English. Combined with Hamcrest matchers, assertions are expressive and failure messages are readable. The combination feels natural once you settle into the syntax.

## Gherkin Syntax

Gherkin introduced a structured way of thinking about test cases: **Given** a starting state, **When** an action is performed, **Then** an expected outcome follows. This maps directly onto the acceptance criteria already written for each user story, which made the transition from specification to test case straightforward.

## Testcontainers

Testing against an in-memory database gives fast feedback but tells you nothing about how the application behaves against a real one. Testcontainers solves this by spinning up a real PostgreSQL instance for each test run. The tests run in CI exactly as they would locally, and the results are meaningful.

## Writing Tests With Purpose

Tests in this project are not written to achieve coverage numbers. They are written to verify that the system behaves correctly according to its specification. The process started with the acceptance criteria and business rules — these were used to derive concrete test cases, and some tests reference a specific user story or business rule directly.

This approach forced a clarity of intent that I found valuable. If a test cannot be traced back to a requirement, it raises the question of what it is actually testing.

## Webhooks and Websockets

This week also briefly covered webhooks and websockets. Webhooks in particular became directly relevant — the CI/CD pipeline later uses a Watchtower webhook to trigger redeployment after a successful build.

## Current State

The test suite covers the core functionality and reflects the business rules and acceptance criteria. The next step is security — authentication and authorization.

{{< article path="devlog-4.md" >}}
{{< article path="devlog-6.md" >}}
