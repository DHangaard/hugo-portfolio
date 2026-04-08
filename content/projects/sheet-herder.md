---
title: "Sheet Herder"
date: 2026-02-06
description: "A REST API backend for managing tabletop RPG characters and campaigns."
tags: ["Sheet Herder", "Java", "REST API", "JPA", "Docker"]
featureimage: sheet-herder-thumbnail.png
draft: false
---

## Project Overview

**Project:** Portfolio & Exam Submission  
**Semester:** DAT 3rd Semester 2026  
**Focus:** Production-ready Java backend with authentication, role-based access control, external API integration, and automated deployment.

Sheet Herder is a backend API for a _Tabletop Roleplaying Game_ (TTRPG) companion application. The system allows players to create and manage their characters and personal notes, while game masters can organise campaigns, track session progress, and keep an eye on their party — all in one place.

---

## Vision

Players manage characters and notes. Game masters create campaigns, track sessions, and oversee their party. One API to power it all.

The longer-term vision is a rules-agnostic companion that supports any TTRPG system. The current implementation is built around the D&D 5e ruleset, using the [D&D 5e REST API](https://www.dnd5eapi.co/) as its reference data source.

---

## Architecture

Sheet Herder is implemented as a layered backend architecture with clear separation between HTTP handling, business logic, and persistence.

The system consists of:

- Javalin controllers handling REST communication
- Service layer implementing domain logic and business rules
- DAO layer using JPA/Hibernate for database access
- PostgreSQL relational data model
- JWT authentication and role-based authorization
- D&D 5e REST API integration for reference data (races, subraces, languages, traits)
- Docker containerisation with Caddy as reverse proxy and automatic TLS
- Automated CI/CD pipeline via GitHub Actions, Docker Hub and Watchtower

---

## Development Log

The project is documented week by week in the devlog series.

{{< article link="/posts/devlog-0/" >}}
{{< article link="/posts/devlog-1/" >}}
{{< article link="/posts/devlog-2/" >}}
{{< article link="/posts/devlog-3/" >}}
{{< article link="/posts/devlog-4/" >}}
{{< article link="/posts/devlog-5/" >}}
{{< article link="/posts/devlog-6/" >}}
{{< article link="/posts/devlog-7/" >}}
{{< article link="/posts/devlog-8/" >}}

---

## Project Video

A short walkthrough of the Sheet Herder portfolio and backend system.

The video covers the project overview, development log, architecture, and a live backend demo.

[Watch on YouTube](placeholder)

---

## Source Code

{{< github repo="DHangaard/sheet-herder" >}}

---

## Live API

The deployed application is available at [sheet-herder-api.dhangaard.dk](https://sheet-herder-api.dhangaard.dk/routes).
