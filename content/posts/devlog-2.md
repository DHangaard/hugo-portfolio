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

Last week the domain was defined and the first entities were in place with basic field mappings. This week the focus shifted to JPA relations, JPQL, and building out the DAO layer. More groundwork than visible output, but necessary groundwork.

## JPA Relations

Adding relations to the entity layer was straightforward once the domain model was in place. The decisions about which associations to model were already made. What required more thought was how to model them.

The default approach was to keep associations unidirectional. If an entity does not need to navigate to another, it should not hold a reference to it. Bidirectional relationships were only introduced where the domain genuinely required navigation in both directions. This keeps coupling low and the codebase easier to reason about.

Cascade behaviour is where business rules translate directly into persistence configuration. BR-6 states that deleting a user removes all their character sheets. On `User`, this maps to `CascadeType.REMOVE` and `orphanRemoval = true` on the `characterSheets` collection:

```java
@OneToMany(mappedBy = "user", cascade = CascadeType.REMOVE, orphanRemoval = true, fetch = FetchType.LAZY)
private List<CharacterSheet> characterSheets = new ArrayList<>();
```

`CascadeType.REMOVE` handles deletion of the parent. When a `User` is deleted, Hibernate removes all associated `CharacterSheet` records automatically. `orphanRemoval = true` handles the additional case where a character sheet is removed from the collection without the user being deleted. Together they cover the business rule completely.

## JPQL

JPQL is Hibernate's query language. Where SQL operates against tables and columns, JPQL operates against the object model — entity class names and field names rather than table and column names. The practical benefit is that queries remain decoupled from the underlying schema.

A simple example from `UserDAO`:

```java
return em.createQuery("""
    SELECT u
    FROM User u
    WHERE LOWER(u.email) = LOWER(:email)
    """, User.class)
    .setParameter("email", email)
    .getResultStream()
    .findFirst();
```

`getResultStream().findFirst()` returns an `Optional` rather than throwing an exception when no result is found, which is the appropriate behaviour for a lookup that may legitimately return nothing. `getSingleResult()` would throw a `NoResultException` on an empty result — useful in some contexts, but not here. The case-insensitive comparison using `LOWER()` on both sides ensures that email lookups are consistent regardless of how the address was entered.

## The DAO Layer

Every entity type gets its own DAO, and every DAO implements a shared interface. `IDAO<T>` defines the base contract:

```java
public interface IDAO<T> {
    T create(T t);
    T getById(Long id);
    T update(T t);
    Long delete(Long id);
}
```

Domain DAOs extend this with operations specific to their entity. `IUserDAO` adds user lookup by email and username, and role management. `ICharacterSheetDAO` adds retrieval by owner:

```java
public interface IUserDAO extends IDAO<User> {
    Optional<User> getByEmail(String email);
    Optional<User> getByUsername(String username);
    User addRole(Long id, Role role);
    User removeRole(Long id, Role role);
}

public interface ICharacterSheetDAO extends IDAO<CharacterSheet> {
    List<CharacterSheet> getAllByUser(User user);
}
```

`UserDAO` is the more complete implementation at this stage. `CharacterSheetDAO` is intentionally sparse. `CharacterSheet` references race, subrace, languages, and ability scores, none of which exist as data yet. The DAO is in place and the contract is defined, but meaningful queries against character sheet content have to wait until the reference layer is populated.

## Scope

As the DAO layer took shape, it became increasingly clear that `Campaign` and `CampaignMembership` would not make it into this exam submission. The domain model accounts for them. The entities exist, the business rules are defined. But building out campaign management would pull focus from the core of what the application needs to demonstrate. That boundary was not drawn in a single moment, but by this point it had become a working assumption. Campaign features are out of scope for this prototype.

## Current State

Relations are mapped, cascade behaviour reflects the business rules, and the core domain DAOs are in place. The reference DAO layer is stubbed and ready, but mostly waiting. The next step is fetching data from the D&D 5e API, which is when the reference layer comes to life and `CharacterSheet` starts to fill out properly.