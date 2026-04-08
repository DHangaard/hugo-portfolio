---
title: "Sheet Herder - Devlog 7"
date: 2026-03-27
description: "CI/CD, GitHub Actions, Docker Hub, Watchtower and getting Sheet Herder live."
categories: ["Devlog"]
tags: ["Sheet Herder", "DevOps", "Docker", "CI/CD"]
series: ["Sheet Herder"]
series_order: 8
draft: false
---

## Where We Left Off

The application is feature-complete for the current scope. This week the focus shifted to deployment — getting Sheet Herder running in a production environment and setting up a pipeline that makes future deployments effortless.

## CI/CD with GitHub Actions

Every push to `main` triggers a GitHub Actions pipeline. The pipeline builds the project, runs the full test suite, and if everything passes, pushes a new Docker image to Docker Hub. If any step fails, nothing is deployed.

This is the kind of feedback loop that changes how you work. Knowing that a passing push is automatically deployed removes the friction between writing code and seeing it live.

## Docker and Caddy

The application runs in Docker. Caddy handles TLS termination and reverse proxying, and obtains SSL certificates automatically. The setup uses separate `docker-compose.yml` files for each concern — Postgres, Caddy, Sheet Herder and Watchtower each have their own stack. This keeps them independently manageable and easy to reason about.

## Watchtower and Webhooks

Watchtower watches the Sheet Herder container and redeploys it when a new image is available. The default polling interval of 60 seconds would have been sufficient, but I chose to implement a webhook trigger instead — partly because near-instant redeployment is a better experience, and partly because working with webhooks was something I had not done before in a deployment context.

The webhook is triggered by the GitHub Actions pipeline via a restricted SSH key after a successful push to Docker Hub. The result is that a passing build is live on the server in under a minute.

## Current State

Sheet Herder is live. The pipeline is working, deployments are automatic, and the infrastructure is stable. The final week is about wrapping up, reflecting, and preparing for the exam.

