---
title: "Sheet Herder - Initial Commit"
date: 2026-02-06
description: "An introduction to this portfolio, its purpose, and the upcoming semester project."
tags: ["Devlog", "Portfolio", "Project", "Sheet Herder"]
draft: false
---

## Initial Commit

To anyone reading, my name is Daniel, and this is the very first blog post on my very first website.

This is all very much outside my comfort zone. I am a private person by nature — I do not use social media, I rarely use my full name publicly, and I am the kind of person who removes name labels from packages and letters before discarding the packaging. Writing publicly, even in a technical context, does not come naturally to me. As an exercise in self-development, I will try to be as honest and personal throughout this site as possible.

This post is not intended to serve as an “About Me” section, but given that this is the starting point of the site, a short introduction feels appropriate.

I am in my mid-thirties and currently enrolled in the Datamatiker programme at Erhvervsakademi København. The English equivalent of this education is an Academy Profession Degree in Computer Science. I am currently on my third semester out of a total of five.

Before starting the Datamatiker programme in January 2025, I had no prior experience with programming. I mention this for context, as this devlog will document not only what I build, but also how I learn, think, and sometimes overthink along the way.

If you would like to know more, you can visit the [About](about) page.


## This Portfolio

During this semester of the programme, we are required to develop a portfolio website. Beyond serving as a learning exercise, the site is also expected to function as a blog where we publish weekly devlogs related to this semester’s portfolio project.

The portfolio is intended to act as a platform where I can present my work, document ongoing projects, and share technical reflections. While a significant part of the content will be related to my studies, the site is not limited to a single project or subject area.

The site itself is mine. Beyond the requirement to publish weekly devlogs as part of the programme, there are no creative or content-related constraints. That freedom allows me to treat this portfolio as something more than an assignment — as a space I can build on over time.

My expectation is that this will become a place where I can express myself through code and technical understanding, and use it as a portfolio that grows alongside me rather than being tied to a single semester.

With that out of the way, it is time to move on to the subject of this post: my semester project.


## Choosing a Project

This semester, we are part of a new exam format and, in a sense, the first test subjects for it. Rather than preparing for isolated exams, we work on a single portfolio project throughout the semester, which later forms the basis for separate backend and frontend exams.

New technologies are introduced gradually and applied directly in the project, making it a continuous learning process rather than a final product-driven one.

During the first week of the semester, we were asked to come up with ideas for this project. That turned out to be more difficult than I expected. I initially had one idea that made sense to me, but I quickly realized that I was starting to fixate on it without properly exploring alternatives.

After a fair amount of thinking — and discarding ideas that felt more like placeholders than real options — I ended up with three concrete project proposals.

### SPLITit

[INSERT SLIDE]

An expense-splitting application where users can create groups, register shared expenses, and automatically calculate who owes whom.

### Sheet Herder

[INSERT SLIDE]

A _Tabletop Roleplaying Game_ (TTRPG) application where game masters can create campaigns, and players can manage characters and notes across sessions.

### What’s Left  

[INSERT SLIDE]

A household application where users can create a home, organize tasks across rooms, and share notes and responsibilities within a family.

After presenting the three ideas to my study group and teacher, the choice ultimately fell on **Sheet Herder**. One of the deciding factors was its strong potential for integration with external APIs, which was something my teacher placed particular emphasis on.


## Expectations

Once the choice fell on Sheet Herder, the mental preparation began almost immediately. Choosing a single project that will shape an entire semester — and form the foundation for both exams — naturally comes with a degree of uncertainty. There is a certain fear of missing out when committing to one idea, knowing that the decision is largely irreversible.

It is not an easy task to select a project that will be examined through technologies I have yet to learn, or fully understand the scope of. Only time will tell whether this project can carry me through the semester, but my intuition tells me it is the right choice.

I expect this to be a project with a relatively simple domain, but with a significant amount of predefined data and rules. That combination can quickly become overwhelming if not handled carefully. Staying within scope will therefore be essential, which places a strong emphasis on preparation and, in particular, a well-defined _Minimum Viable Product_ (MVP).

At the same time, part of the appeal lies in the challenge itself. Of the three project ideas, this is the only one where I cannot yet clearly see the full solution. That uncertainty will likely become a challenge — but it is also what I expect will push my learning the most.


## Current Project State

The project is still in its early stages. Most of my time so far has been spent on careful analysis of the domain, which has resulted in a set of user stories, business rules, and a _Definition of Done_ (DoD). Alongside this, I have worked on a domain model that reflects how I currently view the domain.

[DOMAIN MODEL]

Although the domain itself is relatively simple, arriving at a model that accurately reflects it took more time than expected. One of the key differences compared to previous projects is how roles are not directly tied to a user, but instead depend on context. A user can take on different roles depending on the campaign, which required a shift in how I approached both analysis and modeling.

Another deliberate focus was keeping the model lean. It was an exercise in reducing entities to their essentials and ensuring that the model reflects the actual domain, rather than drifting into TTRPG- or _Dungeons & Dragons_ (D&D)-specific terminology such as classes, species, or similar concepts. The goal was to model what the system needs to support, not the flavor surrounding it.

In the next phase of the project, I will begin setting up the DAO layer. This should provide a clearer picture of how data will be structured, persisted, and accessed, and help clarify how to handle the data needed to support the application’s core functionality: the character sheet.


## Closing Thoughts

This was my first post.

Reading it back, it feels longer than it probably needs to be — but that is fairly typical for me. I usually strive for perfection in most things I do, which in this case resulted in me writing too much out of a nervousness about writing too little.

As mentioned earlier, sharing thoughts like this publicly is not something I am entirely comfortable with. The only way forward is to try things out and adjust along the way. Over time, I expect these devlogs to become more focused and precise as I settle into a format that works better for me.

While this semester project is the main focus right now, it is not the only one I am interested in. The two remaining project ideas are something I plan to return to, and if they are developed further, they will be documented here as well and added to the Projects section so they can be followed in the same way.

Thank you for reading! You can follow the current project status on GitHub:  


{{< github repo="DHangaard/sheet-herder" showThumbnail=true >}}
----