---
title: "Sheet Herder - Devlog 1"
date: 2026-02-13
description: "Domain modelling, business rules, user stories and first steps with JPA."
categories: ["Devlog"]
tags: ["Sheet Herder", "JPA", "Hibernate", "Domain Model"]
series: ["Sheet Herder"]
series_order: 2
draft: false
---

## Where We Left Off

Last week I committed to developing Sheet Herder. Since then, most of the work has been analytical rather than technical. Before writing a single line of persistence code, I wanted to be confident in the domain. That turned out to be the right call.

## Domain Model

The domain model for this project is intentionally simple. There are four core entities: `User`, `CharacterSheet`, `Campaign`, and `CampaignMembership`. The ERD will grow once reference data comes into the picture, but the core has to be right first.

The most interesting design challenge was roles. A user is not simply a player or a game master — they can be both, depending on the campaign. Modelling this as a field on `User` would have been wrong. A user does not have a role. A user has a role *within a campaign*. That distinction forced the introduction of `CampaignMembership` as an explicit entity rather than a simple join table.

`CampaignMembership` maps a user to a campaign, holds their role in that campaign (`GAME_MASTER` or `PLAYER`), and optionally references the character sheet they are participating with. The unique constraint on `(user_id, campaign_id)` enforces at the database level that a user can only hold one membership per campaign.

```java
@Entity
@Table(uniqueConstraints = @UniqueConstraint(
    name = "uk_user_membership_campaign",
    columnNames = {"user_id", "campaign_id"}
))
public class CampaignMembership implements IEntity {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "campaign_id", nullable = false)
    private Campaign campaign;

    @OneToOne(optional = true)
    @JoinColumn(name = "character_sheet_id", unique = true)
    private CharacterSheet characterSheet;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private CampaignRole role;
}
```

`CharacterSheet` is where most of the domain complexity lives. A character belongs to exactly one user, has a name unique within that user's collection, and references reference data — race, subrace, and a set of languages — alongside ability scores and notes stored as element collections.

```java
@Entity
@Table(name = "character_sheet", uniqueConstraints = @UniqueConstraint(
    name = "uk_user_character_name",
    columnNames = {"user_id", "name"}
))
public class CharacterSheet implements IEntity {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "race_id")
    private Race race;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "character_languages",
        joinColumns = @JoinColumn(name = "character_id"),
        inverseJoinColumns = @JoinColumn(name = "language_id")
    )
    private Set<Language> languages;

    @ElementCollection(fetch = FetchType.LAZY)
    @CollectionTable(name = "character_ability_scores", joinColumns = @JoinColumn(name = "character_id"))
    @MapKeyEnumerated(EnumType.STRING)
    @Column(name = "score")
    private Map<Ability, Integer> abilityScores;
}
```

`User` owns character sheets and cascades deletion to them. `Campaign` at this stage is a minimal scaffold — it exists to anchor the relationships and will expand when campaign management comes into scope. Both carry `createdAt` and `updatedAt` fields managed by `@PrePersist` and `@PreUpdate` lifecycle hooks, keeping timestamp logic out of the service layer.

## Reference Data

One part of the domain model that required early scoping decisions was reference data — races, subraces, languages, and traits. The D&D 5e API offers significantly more than this: classes, spells, equipment, and more. Prior research and existing domain knowledge made it straightforward to identify what a character sheet needs at a minimum and what falls beyond that.

For this prototype, the scope is deliberately narrow. Races, subraces, languages, and traits cover the core of character creation without overcomplicating the integration work. Classes, spells, and the rest are not out of reach — the domain model is designed to accommodate them — but they are out of scope for now. That boundary was drawn consciously; scope management rather than limitation.

## Lombok and JPA Equality

One issue that came up while mapping entities: Lombok's `@EqualsAndHashCode` annotation generates `equals` and `hashCode` methods based on the fields of a class. That sounds reasonable until you factor in how Hibernate works under the hood.

When Hibernate loads an associated entity lazily — deferring the database call until the data is actually accessed — it does not give you the real object immediately. Instead, it gives you a placeholder that looks and behaves like the real one but is essentially an empty wrapper until it is touched. The problem is that Lombok's generated `equals` compares class types directly, and this placeholder's class is not the same as the actual entity class. Two objects that represent the same database record can therefore compare as unequal, which causes subtle and hard-to-trace bugs in collections, caches, and anywhere else equality matters.

The correct approach is to resolve what the real underlying class is before comparing — and to base equality solely on the database identifier rather than object state. This is not something I arrived at independently; it is what IntelliJ generates when asked to produce `equals` and `hashCode` for a JPA entity, and I added a comment to each entity to make that explicit. All domain entities in this project use this pattern:

```java
@Override
public final boolean equals(Object object) {
    if (this == object) return true;
    if (object == null) return false;
    Class<?> oEffectiveClass = object instanceof HibernateProxy
        ? ((HibernateProxy) object).getHibernateLazyInitializer().getPersistentClass()
        : object.getClass();
    Class<?> thisEffectiveClass = this instanceof HibernateProxy
        ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass()
        : this.getClass();
    if (thisEffectiveClass != oEffectiveClass) return false;
    CampaignMembership that = (CampaignMembership) object;
    return getId() != null && Objects.equals(getId(), that.getId());
}

@Override
public final int hashCode() {
    return this instanceof HibernateProxy
        ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass().hashCode()
        : getClass().hashCode();
}
```

An unpersisted entity — one with no `id` yet — is not equal to anything, including itself. That is intentional. Until a record exists in the database, there is no identity to compare.

## Business Rules

Defining business rules had more impact on the design than expected. They establish the boundaries of the system — what is allowed, what is not, and who is responsible for what. Getting them right early made later decisions significantly easier to reason about.

A few rules shaped the persistence layer directly. Two of them define cascade deletion behaviour: deleting a user removes all their character sheets; deleting a user who owns a campaign removes the campaign and all associated data. These rules directly inform the Hibernate cascade configuration — the domain rules and the persistence behaviour have to match. A business rule that is not enforced at the database level is not really a rule.

Another rule states that a character cannot participate in multiple campaigns simultaneously. This is reflected in the `unique = true` constraint on `character_sheet_id` in `CampaignMembership`. The uniqueness is not just an application-level check — the database enforces it.

The permission rules define visibility boundaries that will drive authorization logic later. A game master can see full character details for all characters in their campaign. A player sees only limited public information for others. Character notes are entirely private, even from the game master.

## User Stories and Definition of Done

The user stories are intentionally broad. As a solo contributor, fully decomposed tasks add overhead without adding value. They define scope and serve as the basis for acceptance criteria, which will drive how tests are written later.

One observation from writing them: the stories map naturally to roles. The player-facing stories cover authentication, character creation and management, and notes. The game master stories cover campaign creation, membership management, and session logs. There is also a cross-cutting story about reference data access during character creation. That split reflects the domain well and makes it straightforward to reason about access control when the time comes.

The Definition of Done is short and deliberate — an internal agreement that keeps scope honest. The most important item is the last: a feature must integrate without breaking what already works. In a project built bottom-up, that means every layer has to be verifiable before the next is built on top of it.

## Java Persistence API

This week introduced JPA with a focus on ORM, entity mapping, and DAOs. Coming from a solid understanding of JDBC and raw SQL, the abstraction layer took some adjustment. The mental shift from writing queries to mapping objects is significant, and I spent time understanding what Hibernate was doing under the hood rather than treating it as a black box. That paid off almost immediately when the equality issue came up.

The implementation at this stage is intentionally lean — basic field mappings, no relations yet. Relations are next, and they will be the first real test of whether the domain model holds up in practice.

## Current State

The domain is defined, the rules are documented, and the first entities are in place. The next step is JPA relations, cascade behaviour, and fetch types — which will determine how well the domain model translates into a working persistence layer.