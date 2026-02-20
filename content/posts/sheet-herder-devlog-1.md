---
title: "Sheet Herder - Devlog 1"
date: 2026-02-13
description: "Input description"
category: ["Devlog"]
tags: ["Sheet Herder"]
draft: true
---

## Where we left off

Last week I committed to developing Sheet Herder - A TTRPG Companion App. I began

### Hugo and Markdown

A lot has happened with my understaning of Hugo from last week until now. Some might say i fell into a rabbit hole. This shall not be the main focus of this post, but I need to address some points so that my style of writing and this page as a whole makes more sense.
As of writing this, I am currently migrating Hugo themes Blowfish to PaperMod. This blogpost wil be posted on the first iteration of my site, and other than my devlogs and blogposts, i will not add more to this site before i have updated the theme. 

There are a couple of reasons for changing themes:
: I do not find the look of Blowfish to my liking (even with it's modularity). 
: I find Blowfish *too beginner-friendly*, creating too much boilerplate and limiting my conceptual understanding of Hugo
: I really like the look of PaperMod

The rabbit hole took me places and one of them was [The Markdown Guide](https://www.markdownguide.org/ "www.markdownguide.org"). I have decided to try and master the *Art of Markdown* with these devlogs, and my README's, as my training grounds. So if you think that I use a lot of Markdown syntax, you are correct - I am ***practising***!

I plan on doing a seperate devlog revolving around this site when the theme gets migrated, and the repository containing the site will also appear in [Projects](/projects "/projects").  

### Domain Model

The domain model for this project is very simple but also a clean depiction of how I view the Domain. Do not be fooled! The *Entity Relationship Diagram* (**ERD**) and class diagram will be much more complex. As of now, I will work on entities that reflect those present in this model, but as soon as we start working with integrating API's, the character sheet will expand (***a lot***!).

[Insert model]
```uml
@startuml

class Campaign
{
}

class CampaignMembership
{
role : CampaignRole
}

class Character
{
}

ENUM CampaignRole
{
GAME_MASTER
PLAYER
}

class User
{
}

User "1" --- "0..*" Character : has
User "1" --- "0..*" CampaignMembership : participates as

CampaignMembership "1..*" --- "1" Campaign : belongs to
Character "0..*" --- "0..1" Campaign : participates in

@enduml
```

### Business Rules

Deciding on business rules was a important design choice that had more impact than I initially thought. When pitching and designing my idea, I kept thinking that Sheet Herder would be a rules-agnostic TTRPG companion app with the main focus being on the Dungeons & Dragons ruleset.
This changed ... 
While still the ambition, it is simply not within scope of the mere 10 weeks I have to develop my backend, and I ultimately ended up designing around the D&D ruleset. 
I managed to still keep the core idea of the application, which was to add an alternative to [D&D Beyond](https://www.dndbeyond.com/ "www.dndbeyond.com") where one could host a game, using rulebooks, without having to pay ekstra for licensed content in the app. 
This is reflected in few of the business rules, but most of them are defined so that the project can implemented other rulesets in a later stage. Although happy with the state of these rules AS-IS, some of them might need a little tweaking to be [NOT ALLOW MISUNDERSTANDING]

```markdown
# Business Rules
This document defines the core domain rules that apply across the application, independent of specific user flows.

## Character
- **BR-1**: A character belongs to exactly one user (the creator)
- **BR-2**: Only the character owner can modify character details
- **BR-3**: A character cannot participate in multiple campaigns simultaneously
- **BR-4**: Only the character owner can delete a character
- **BR-5**: When a character leaves a campaign, the character remains owned by the player

## Campaign
- **BR-6**: A campaign has exactly one Game Master
- **BR-7**: A campaign can have 1–8 players (configurable by the Game Master)
- **BR-8**: Only the Game Master can invite or remove players
- **BR-9**: Players may leave a campaign freely; the Game Master cannot leave their own campaign
- **BR-10**: Game Masters can delete campaigns they own, which removes all associated campaign data
- **BR-11**: Only the Game Master can modify campaign settings
- **BR-12**: Only characters owned by campaign participants can be assigned to a campaign

## Campaign Documentation
- **BR-13**: Campaign notes are private to the Game Master and are not visible to players
- **BR-14**: Session logs are created by the Game Master and are readable by all campaign participants

## Permissions
- **BR-15**: Game Masters can view full mechanical details of all characters in their campaigns
- **BR-16**: Players can view limited public information of characters in shared campaigns
- **BR-17**: Character notes are private and visible only to the character owner
- **BR-18**: Game Masters cannot view or modify player-created character notes
```

### User Stories

I am satisfied with how the user stories turned out. [SOMETHING ABOUT THE USER STORIES]
Here are some of the user stories that  [DEFINES THE CORE OF THE PROGRAM]

```markdown
# User Stories
This document outlines the high-level user stories that define the functional scope of the application.<br>
The stories are intentionally broad and represent core use cases rather than fully decomposed tasks as the project is developed by a single contributor.<br>


```

### Definition of Done

Even though my *Definition of Done* (**DOD**) seems short, I felt I needed it as an internal agreement - One that would make me accountable and not loose focus [ADD MORE]

```markdown
# Definition of Done

A user story is considered done when:

- Core functionality is implemented according to acceptance criteria
- Input validation and basic error handling are implemented
- Authorization rules are enforced where applicable
- The feature is covered by relevant tests
- Code is self-describing where possible, with comments added for complex domain logic or non-obvious implementation choices
- The feature is integrated into the application without breaking existing functionality
```

## Java Persistence API

In the past two weeks we have learned about *Java Persistence API* (**JPA**) with focus on:
- *Object Relational Mapping* (**ORM**)
- JPA
- Hibernate
- *Java Persistence Query Language* (**JPQL**)
- *Data Access Objects* (**DAO**)
- `Lombok`
- JPA Relations
    - One-To-One
    - One-To-Many
    - Many-To-Many

Having a good understanding of *Java Database Connectivity* (**JDBC**) and SQL ...
As mentioned before, I will implment this with entities that reflects the ... 

## Implementation

When implementing strive to be to the point, and only implement what is needed right now, one step at the time. This helps me from creating *"imaginary problems"* or problems that *might* turn out to be a problem later. If they do, I will deal with them then. This is different from how I usually work, where I want every solution to be perfect. While this creates solid code, it can very easily turn a simple problem complex and make the project go out of scope or [TAKE MORE TIME THAN PLANNED?]
[WHAT HAVE I DONE IN THIS IMPLEMENTATION]
[TDD]
Kent Beck described the TDD workflow  in his recent article [Canon TDD](https://tidyfirst.substack.com/p/canon-tdd "tidyfirst.substack.com") 

## Current state
I would have liked to be further with Sheet Herder that where it is in it's current state but I would not have prioritized my time differently. Doing homework, class assignments and reading up on the subjects at hand left me with a good understanding of JPA - It also had me coding JPA implmentation that I might not need in this specific project which makes my time spent well. 

Sheet Herder is in a good state as is. I have to fight the desire to create more than is needed when I know that I do not yet understand all the technologies that needs to be implemented. 

[INSERT CURRENT STATE]

```text
sheet-herder/
├─ pom.xml
├─ src/
│  ├─ main/
│  │  ├─ java/
│  │  │  └─ app/
│  │  │     ├─ Main.java
│  │  │     ├─ config/
│  │  │     │  ├─ EntityRegistry.java
│  │  │     │  ├─ HibernateBaseProperties.java
│  │  │     │  ├─ HibernateConfig.java
│  │  │     │  └─ HibernateEmfBuilder.java
│  │  │     ├─ daos/
│  │  │     │  ├─ IDAO.java
│  │  │     │  └─ StudyDAO.java
│  │  │     ├─ entities/
│  │  │     │  └─ Study.java
│  │  │     ├─ enums/
│  │  │     │  └─ StudyPhase.java
│  │  │     ├─ exceptions/
│  │  │     │  └─ ApiException.java
│  │  │     └─ utils/
│  │  │        └─ Utils.java
│  │  └─ resources/
│  │     ├─ config.properties
│  │     └─ logback.xml
│  └─ test/
│     └─ java/
│        └─ app/
│           ├─ config/
│           │  └─ HibernateTestConfig.java
│           ├─ daos/
│           │  └─ StudyDAOTest.java
│           └─ testutils/
│              └─ StudyTestPopulator.java
```

## Whats next?

### Password hashing with Argon2

### D&D API Integration
