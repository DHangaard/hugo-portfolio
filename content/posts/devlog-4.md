---
title: "Sheet Herder - Devlog 4"
date: 2026-03-06
description: "REST principles, HTTP methods, status codes, logging and building the API layer with Javalin."
categories: ["Devlog"]
tags: ["Sheet Herder", "REST", "Javalin", "Logging"]
series: ["Sheet Herder"]
series_order: 5
showSummary: true
draft: false
---

## Where We Left Off

The persistence layer is in place and reference data is being fetched and synced at startup. This week the focus shifted to the API layer — the part of the application that the outside world actually talks to.

## REST Principles

This week started with the guiding principles behind RESTful API design. A few concepts stood out as particularly important for how I approached the implementation.

**Idempotence** is worth pausing on. An operation is idempotent if calling it multiple times produces the same result as calling it once. `GET`, `PUT` and `DELETE` are idempotent — `POST` is not. This distinction matters when designing endpoints, especially when thinking about what happens if a request is retried.

**Resource naming** is another area where REST has strong conventions. URIs identify resources, not actions. `/character-sheets/{id}` is a resource. `/getCharacterSheet?id=1` is not. Following these conventions makes the API predictable and easier to consume.

## HTTP Methods and Status Codes

Using the right status code is part of the contract between the API and its consumers. A `404` means the resource was not found. A `403` means the resource exists but the caller is not allowed to access it. A `409` means there is a conflict — for example, a character name that already exists for that user. Getting these right matters.

## Logging

This week also introduced Logback and SLF4J. Rather than the minimal setup covered in class, I spent time configuring a logging setup that would be useful in practice — separate appenders for application logs and error logs, rolling policies, and environment-driven log levels. The health check endpoint is excluded from the request logger to avoid Docker polling noise.

## Javalin

Building the API layer with Javalin was straightforward once the architecture was in place. Controllers handle HTTP concerns, services own the business logic, and the two are connected through the route definitions. Javalin's exception handling integrates cleanly with the custom exception hierarchy already in place.

## Current State

The core API endpoints are in place. The next step is testing — making sure the API behaves correctly under the conditions defined by the acceptance criteria and business rules.

{{< article link="/posts/devlog-3/" >}}
{{< article link="/posts/devlog-5/" >}}
