---
title: "Sheet Herder - Devlog 3"
date: 2026-02-27
description: "Fetching reference data from the D&D 5e API, threading with ExecutorService and mapping with DTOs."
categories: ["Devlog"]
tags: ["Sheet Herder", "API Integration", "ExecutorService", "DTO"]
series: ["Sheet Herder"]
series_order: 4
draft: false
---

## Where We Left Off

The DAO layer is in place and the domain entities are mapped. This week the focus shifted outward — fetching reference data from an external API and integrating it into the persistence layer.

## The D&D 5e API

The D&D 5e REST API provides all the reference data needed for character creation. The decision to fetch this data at startup and persist it locally was deliberate. It avoids a runtime dependency on an external service and gives the application full control over the data it works with. If the API is unavailable after the first run, the application still functions.

Fetching on every startup is a somewhat redundant approach — in a production setting, a scheduled job or a webhook triggered by upstream data changes would be the appropriate design. I am aware of that. The startup fetch is kept here because it is predictable, easy to reason about, and honest about where my skills are right now. It is a design choice I can explain and defend, not one I arrived at by accident.

The integration follows a two-step pattern. A first request fetches a summary list — names and URLs, nothing more. A second round of requests fetches the full detail for each entry. For languages, that looks like this:

```java
// Step 1: summary list
public record DNDLanguageResultDTO(
    @JsonProperty("count") int count,
    @JsonProperty("results") List<DNDLanguageDTO> languages
) {}

// Step 2: summary entry with URL for detail fetch
public record DNDLanguageDTO(
    @JsonProperty("index") String index,
    @JsonProperty("name") String name,
    @JsonProperty("url") String url
) {}

// Step 3: full detail
public record DNDLanguageDetailDTO(
    @JsonProperty("name") String name,
    @JsonProperty("desc") String description,
    @JsonProperty("type") String type,
    @JsonProperty("typical_speakers") List<String> typicalSpeakers,
    @JsonProperty("script") String script
) {}
```

`@JsonIgnoreProperties(ignoreUnknown = true)` is applied to all external DTOs. The API returns more fields than the application needs, and ignoring unknown properties means the mapping does not break if the API adds or changes fields.

The separation between summary and detail DTOs is intentional. The domain model should not be shaped by the structure of an external API. These DTOs model the external response exactly as it arrives. The service layer is responsible for mapping them into domain entities before anything is persisted.

## Concurrent Fetching

Fetching detail records sequentially would mean one HTTP request per entry, one after another. For a dataset the size of D&D traits — over a hundred entries — that adds up. Instead, each detail fetch is submitted as a concurrent task and the results are collected once all are complete:

```java
List<Callable<DNDLanguageDetailDTO>> tasks = resultDTO.languages().stream()
    .map(language -> {
        String url = String.format(BASE_URL, language.url());
        return (Callable<DNDLanguageDetailDTO>) () -> dndClient.fetchLanguageDetails(url);
    })
    .toList();

return ThreadUtil.fetchConcurrently(tasks);
```

`ThreadUtil.fetchConcurrently` submits all tasks to a fixed thread pool, collects the futures, and waits for each result:

```java
public static <T> List<T> fetchConcurrently(List<Callable<T>> tasks) {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    List<T> result = new ArrayList<>();

    List<Future<T>> futures = tasks.stream()
        .map(executorService::submit)
        .toList();

    try {
        for (Future<T> future : futures) {
            result.add(future.get());
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new ConcurrentExecutionException("Failed to fetch data concurrently", e);
    } catch (ExecutionException e) {
        throw new ConcurrentExecutionException("Failed to fetch data concurrently", e);
    } finally {
        executorService.shutdown();
    }

    return result;
}
```

The difference in startup time is noticeable. More importantly, it introduced a pattern worth understanding: submit tasks, collect futures, handle results. The executor is always shut down in the `finally` block regardless of outcome.

## Content Hashing and Sync

Fetching reference data at startup raises a question: what happens on subsequent startups? Deleting and reinserting every record on each run would be wasteful and would break any foreign key relationships that have built up. The approach taken is to detect changes and only write what has actually changed.

Each reference entity stores a `contentHash` field — a SHA-256 hash computed from its fields at the time it was fetched. On every startup, the incoming hash is compared against the stored one. If they match, the record is skipped. If they differ, it is updated. If no record exists yet, it is inserted:

```java
private Language resolveLanguage(EntityManager em, Language incoming, Language existing) {
    if (existing == null) {
        em.persist(incoming);
        return incoming;
    }
    if (incoming.getContentHash().equals(existing.getContentHash())) {
        return existing;
    }
    return applyUpdate(existing, incoming);
}
```

The hash itself is computed from a normalised string representation of the fields that matter:

```java
public static String sha256Hex(String input) {
    MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
    byte[] hashBytes = messageDigest.digest(input.getBytes(StandardCharsets.UTF_8));
    return toHex(hashBytes);
}
```

This makes the sync operation idempotent. Running it once or a hundred times produces the same result. The hash is the source of truth for whether a record needs updating, not a field-by-field comparison.

## Fetch Types

With reference data now in the application, the fetch type decisions made earlier became immediately testable. The first attempt to load a persisted `Race` and display it in the terminal produced a `LazyInitializationException` on the associated languages and traits.

The cause is a mismatch between when Hibernate loads data and when the session is open. With `FetchType.LAZY`, associations are only fetched while the `EntityManager` is still open. Once it is closed, any attempt to access an unloaded association fails. For reference data, this is a poor fit. Races, subraces, languages, and traits are always needed together. Deferring their loading adds complexity with no real benefit.

The decision was to switch all reference entity associations to `FetchType.EAGER`. Everything loads upfront in a single operation. For a dataset of this size and access pattern, this feels like the right tradeoff — at least one I can reason about and defend.

Even with `EAGER` configured, there is a subtlety worth addressing. By default, Hibernate loads eager associations with separate queries — one for the parent record, one for each associated collection. For a list of races, that means one query for the races themselves, then additional queries for each race's languages and traits. The number of queries grows with the number of records.

`LEFT JOIN FETCH` in JPQL overrides this and loads everything in a single query:

```java
return em.createQuery("""
    SELECT DISTINCT r
    FROM Race r
    LEFT JOIN FETCH r.languages
    LEFT JOIN FETCH r.traits
    """, Race.class)
    .getResultList();
```

`SELECT DISTINCT` is necessary because the join produces one row per combination of race, language, and trait. Without it, the same race appears multiple times in the result list.

## Current State

Reference data is fetched at startup, persisted locally, and kept current through hash-based change detection. The character sheet now has something meaningful to reference. The next step is building the REST API layer on top of what has been built so far.