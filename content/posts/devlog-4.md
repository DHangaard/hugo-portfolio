---
title: "Sheet Herder - Devlog 4"
date: 2026-03-06
description: "REST principles, HTTP methods, status codes, logging and building the API layer with Javalin."
categories: ["Devlog"]
tags: ["Sheet Herder", "REST", "Javalin", "Logging"]
series: ["Sheet Herder"]
series_order: 5
draft: false
---

## Where We Left Off

Reference data is fetched, persisted, and synced at startup. This week the focus shifted to the API layer — the part of the application the outside world actually talks to.

## REST Principles

This week started with the guiding principles behind RESTful API design. A few concepts stood out as particularly important for how I approached the implementation.

Idempotence is worth pausing on. An operation is idempotent if calling it multiple times produces the same result as calling it once. `GET`, `PUT`, and `DELETE` are idempotent. `POST` is not. This distinction matters when designing endpoints, especially when thinking about what happens if a request is retried.

Resource naming is another area where REST has strong conventions. URIs identify resources, not actions. `/character-sheets/{id}` is a resource. `/getCharacterSheet?id=1` is not. Following these conventions makes the API predictable and easy to consume.

## Javalin and the API Layer

The API layer is built with Javalin, a lightweight Java web framework. Each concern in `ApplicationConfig` is handled by a dedicated private method — routing, exception handling, JSON serialisation, and logging each have a clear home.

The startup sequence reflects this structure:

```java
public static Javalin startServer(int port) {
    ExecutionTimer.start();
    JWTUtil.validate();
    DIContainer diContainer = DIContainer.getInstance();
    DataSeeder.seed(diContainer);
    Routes routes = buildRoutes(diContainer);

    Javalin app = Javalin.create(config -> {
        configureRoutes(config, routes);
        configureExceptions(config);
        configureJackson(config, diContainer);
        configureLogger(config);
    }).start(port);

    ExecutionTimer.finish("SheetHerder ready on port " + port);
    return app;
}
```

`ExecutionTimer` wraps the entire startup process. Split points are recorded at meaningful milestones — after reference data is fetched, after it is synchronised — and the total time is logged when the server is ready. In practice this looks something like:

```
Reference data fetched: 00:01.24 (total: 00:01.24)
Reference data synchronized: 00:00.43 (total: 00:01.67)
SheetHerder ready on port 7070: 00:02.14
```

This makes slow startups immediately visible rather than something you notice by chance.

## Dependency Injection

`DIContainer` is a manually managed singleton that instantiates and wires together every component in the application — DAOs, services, controllers, and the HTTP client. There is no framework handling this. Every dependency is constructed explicitly and injected through constructors.

This is a deliberate choice. With a manual container, the dependency relationships are visible and explicit. At exam time, being able to explain exactly how a controller gets its service, and how that service gets its DAO, is straightforward because there is nothing implicit about it.

## Routes and Controllers

Routes define the endpoints. Controllers handle the HTTP concerns and delegate to services. The two are kept deliberately separate.

`LanguageRoute` shows the pattern:

```java
public class LanguageRoute {
    private final IReferenceController languageController;

    protected EndpointGroup getRoutes() {
        return () -> path("languages", () -> {
            get(languageController::getAll);
            get("{id}", languageController::getById);
            get("name/{name}", languageController::getByName);
        });
    }
}
```

`LanguageController` handles the HTTP layer and nothing else:

```java
public class LanguageController implements IReferenceController {

    @Override
    public void getById(Context ctx) {
        Long id = Long.parseLong(ctx.pathParam("id"));
        LanguageDTO languageDTO = languageService.getById(id)
            .orElseThrow(() -> new NotFoundException("Language with id " + id + " not found"));
        ctx.status(HttpStatus.OK).json(languageDTO);
    }

    @Override
    public void getAll(Context ctx) {
        ctx.status(HttpStatus.OK).json(languageService.getAll());
    }
}
```

The controller parses the request, calls the service, and writes the response. Business logic does not belong here, and there is none. Both route and controller implement interfaces — `IReferenceController` defines the contract, and every reference type gets its own implementation. Adding a new reference type means implementing the interface, not redesigning the layer.

## Exception Handling

Exceptions in this application are purpose-specific. `NotFoundException`, `ValidationException`, and `ConflictException` all extend `ApiException`, which carries an HTTP status code alongside the message. When something goes wrong, the exception already knows what status code to return.

`ApplicationConfig` maps these to HTTP responses in one place:

```java
config.routes.exception(ApiException.class, (e, ctx) -> {
    log.warn("{} {} - {}", ctx.method(), ctx.path(), e.getMessage());
    ctx.status(e.getCode())
        .json(Map.of(
            "status", e.getCode(),
            "message", e.getMessage()
        ));
});

config.routes.exception(NumberFormatException.class, (e, ctx) -> {
    log.warn("{} {} - Invalid number format: {}", ctx.method(), ctx.path(), e.getMessage());
    ctx.status(HttpStatus.BAD_REQUEST.getCode())
        .json(Map.of(
            "status", HttpStatus.BAD_REQUEST.getCode(),
            "message", "Invalid ID format: expected a number"
        ));
});

config.routes.exception(Exception.class, (e, ctx) -> {
    log.error("{} {} - Unhandled exception", ctx.method(), ctx.path(), e);
    ctx.status(HttpStatus.INTERNAL_SERVER_ERROR.getCode())
        .json(Map.of(
            "status", HttpStatus.INTERNAL_SERVER_ERROR.getCode(),
            "message", "Internal server error"
        ));
});
```

`NumberFormatException` is handled explicitly because `Long.parseLong` in `getById` will throw it if a caller passes a non-numeric id. Without this handler, a request to `/languages/abc` would fall through to the generic 500 handler, which would be the wrong response for what is ultimately a bad request.

Any other unhandled exception returns a generic 500 with no internal details exposed. The full stack trace is logged, but the response tells the caller only that something went wrong. Internal errors should be visible to the developer through logs, not to the caller through the API response.

## Logging

Logging is configured with Logback and SLF4J. The configuration grew beyond what was strictly necessary, partly because I found it genuinely interesting to work through.

The log level is environment-driven — `${LOG_LEVEL:-INFO}` defaults to `INFO` but can be overridden at runtime without rebuilding. This matters in a deployed environment where you might want to temporarily raise verbosity without touching the application.

Two separate `RollingFileAppender` instances write to different files for different purposes. The application log captures everything at the configured level and above. The error log captures only `ERROR` level and above, kept separately so errors are easy to find without scanning through general output. Both files are compressed to `.gz` on rollover, which keeps disk usage manageable over time.

The rolling policies differ deliberately. Application logs roll daily and are kept for 30 days with a 1GB total size cap — frequent enough to keep individual files small, with enough history to trace recent behaviour. Error logs roll weekly and are kept for 13 weeks, roughly a quarter. Errors are less frequent but more important to retain, and a weekly file makes it easy to correlate errors with a specific period.

Third-party frameworks are capped at `WARN` to avoid log noise from Hibernate, Testcontainers, Jetty, and HikariCP. HikariCP is the one exception — connection pool startup is logged at `INFO` because it is useful to see during startup.

The request logger excludes the health check endpoint:

```java
config.requestLogger.http((ctx, ms) -> {
    if (ctx.path().equals("/api/v1/health-check")) return;

    if (ctx.status().getCode() >= 500) {
        log.error("{} {} - {} ({}ms)", ctx.method(), ctx.path(), ctx.status(), ms.longValue());
    } else if (ctx.status().getCode() >= 400) {
        log.warn("{} {} - {} ({}ms)", ctx.method(), ctx.path(), ctx.status(), ms.longValue());
    } else {
        log.info("{} {} - {} ({}ms)", ctx.method(), ctx.path(), ctx.status(), ms.longValue());
    }
});
```

Docker health checks poll the endpoint every few seconds. Without the exclusion, those requests would flood the logs and make it harder to see what is actually happening.

## Current State

The core API endpoints are in place. The next step is testing — making sure the API behaves correctly under the conditions defined by the acceptance criteria and business rules.