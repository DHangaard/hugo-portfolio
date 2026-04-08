---
title: "Sheet Herder - Devlog 6"
date: 2026-03-20
description: "BCrypt, JWT, authentication, authorization and building a security layer that holds up."
categories: ["Devlog"]
tags: ["Sheet Herder", "Security", "JWT", "BCrypt"]
series: ["Sheet Herder"]
series_order: 7
draft: false
---

## Where We Left Off

The API is tested and the core functionality is working. This week the focus shifted to security — which in a backend application means two things: who you are, and what you are allowed to do.

## Password Hashing with BCrypt

BCrypt is the standard choice for password hashing, and for good reason. It is slow by design — the cost factor controls how computationally expensive the hash is, making brute force attacks progressively less practical as the cost is raised.

One detail worth addressing: BCrypt has a 72-byte input limit. A password longer than 72 bytes is silently truncated, which means two different passwords could produce the same hash. The solution is to prehash the password with SHA-256 before passing it to BCrypt — this produces a fixed-length Base64-encoded string regardless of the original password length, ensuring full entropy is preserved.

## JWT Authentication

Authentication is implemented using JWT tokens. On login, a signed token is issued containing the user's identity and roles. The token must be included in subsequent requests and is validated on every protected endpoint.

The implementation uses Nimbus JOSE+JWT — a well-maintained library that handles the cryptographic details. The wrapper around it is kept minimal and focused: sign a token, validate a token, extract claims.

A detail I am particularly satisfied with: missing `JWT_SECRET` or `JWT_ISSUER` environment variables cause the application to crash immediately on startup. A missing configuration should surface as an explicit error at startup, not as a cryptic failure at runtime.

## Role-Based Authorization

Authorization is enforced through Javalin before-filters. Every route declares its required role explicitly. Public endpoints use `Role.ANYONE` — not the absence of a role check, but an explicit statement of intent. This makes it immediately clear when reading a route definition whether authentication is required.

## Current State

The security layer is in place. Authentication works, authorization is enforced, and passwords are handled correctly. The next step is deployment — getting the application running in a production environment.
