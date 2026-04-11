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

1. [Devlog 0 - Initial Commit](/posts/devlog-1.md)<br/>
2. [Devlog 1 - Domain modelling, business rules, user stories and first steps with JPA](/posts/devlog-1.md)<br/>
3. [Devlog 2 - JPA relations, fetch types and the first real test of the domain model](/posts/devlog-2.md)<br/>
4. [Devlog 3 - Fetching reference data from the D&D 5e API, threading with ExecutorService and mapping with DTOs](/posts/devlog-3.md)<br/>
5. [Devlog 4 - REST principles, HTTP methods, status codes, logging and building the API layer with Javalin](/posts/devlog-4.md)<br/>
6. [Devlog 5](/posts/devlog-5.md)<br/>
7. [Devlog 6](/posts/devlog-6.md)<br/>
8. [Devlog 8](/posts/devlog-7.md)<br/>
9. [Devlog 9](/posts/devlog-8.md)<br/>

---

## Project Video

A short walkthrough of the Sheet Herder portfolio and backend system.

The video covers the project overview, development log, architecture, and a live backend demo.

[Video Demo](https://youtu.be/hG4LAV34rLU)

---

## Source Code

{{< github repo="DHangaard/sheet-herder" >}}

---

## Live API

The deployed application is available at [sheet-herder-api.dhangaard.dk](https://sheet-herder-api.dhangaard.dk/routes).
