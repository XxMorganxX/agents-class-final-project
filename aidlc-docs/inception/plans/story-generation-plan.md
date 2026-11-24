# Story Generation Plan

**Stage:** INCEPTION - User Stories (Part 1: Planning)
**Project:** Anti-Vibe Coding (Pi, the learning sidecar)
**Inputs read:** `inception/requirements/requirements.md` (approved 2026-11-23), `inception/plans/user-stories-assessment.md`, `docs/founding-idea.md`, `aidlc-state.md`
**Outputs (Part 2):** `inception/user-stories/stories.md`, `inception/user-stories/personas.md`

## How to use this file

1. **Answer the questions** in the Questions section below. Write the letter of your choice after each `[Answer]:` tag. If no option fits, pick the last option (Other) and describe what you want. You can add a short note after any letter, for example `[Answer]: B, but keep it under 30 stories`.
2. Answer every question. The answers decide how the stories are cut, written and checked, and Units Generation maps every story to a unit later.
3. Tell the agent when you're done. The agent checks the answers for gaps and contradictions, and may write a short clarification file.
4. **Approve the plan,** or ask for changes. Only then does Part 2 (writing the stories and personas) start.

The checklist in the Execution Plan section below fills in once the questions are answered. Steps that depend on an answer say which one.

## What this plan leaves alone

Requirements.md already decides these points, so no question reopens them:

- Who the users are: a CS student at the intro-to-intermediate level, and a teammate running the demo (§4).
- What Pi does and doesn't do: the FR and NFR lists, the nine lesson component types, and the out-of-scope list (§8).
- The demo app (a WebSocket chat server) and acceptance criteria AC-1 to AC-6.

The questions below are about **how to turn those requirements into stories**: which personas, how to cut and group stories, how to write acceptance criteria, and how to handle the parts of Pi that the student never sees directly.

---

# Story breakdown approaches

The rules ask the plan to lay out the main ways to organize the stories. Question 4 picks one.

| Approach | How stories are grouped | Strengths for Pi | Weaknesses for Pi |
|---|---|---|---|
| **User journey** | By the order a student meets things: set up, pair with Claude Code, learn while Claude builds, answer exercises, recover from wrong answers, end the session | Reads like the demo. Makes gaps in the flow obvious (what happens between "wrong answer" and "retest"?) | Shared machinery, such as the component registry or the caps, has no natural home and gets split across stages |
| **Feature** | By requirement area: FR-0 agent, FR-1 pairing, FR-2 components, and so on | One-to-one with requirements.md, so traceability is easy | Reads like a spec, not a user's experience. Stories drift toward "the system shall" |
| **Persona** | By user: student stories, then demo-teammate stories | Shows clearly who each story serves | Most stories belong to the student, so one group swallows nearly everything |
| **Domain** | By area of knowledge: teaching, grading, setup and pairing, cost and safety | Groups match likely code modules, which helps Units Generation | Splits one student moment (answer, grade, drill-down) across domains |
| **Epic** | Epics with smaller stories underneath. The epics can follow any of the approaches above | Gives two levels: the big picture and testable small stories. Epics can line up with units later | Adds a layer to maintain. Weak epics just rename sections |

A **hybrid** is allowed, for example journey-based epics with feature-sized stories inside them. If you choose one, say which approach sets the epics and which sets the stories.

---

# Questions

## Part A: Personas

## Question 1
Which personas should `personas.md` contain?

*Why we ask:* Requirements §4 names a primary user (a student at the "intro-to-intermediate" level) and a secondary user (a teammate running the demo). "Intro-to-intermediate" is a wide range. A student in CS1 and a junior reading WebSocket code need different things from the same feature: one needs every term defined, the other wants Pi to skip what they know. Splitting the student persona makes stories cover both ends. Keeping one keeps the set small.

A) Two personas: one student at the intermediate level, and the demo teammate

B) Three personas: an intro-level student, an intermediate student, and the demo teammate

C) Three personas: one student, the demo teammate, and a course instructor who recommends Pi to students (no stories of their own; used only to explain why the session summary metrics matter)

D) Four personas: intro student, intermediate student, demo teammate, and course instructor as in C

E) Other (please describe after [Answer]: tag below)

[Answer]: B. Two student levels (intro and intermediate) plus the demo teammate. Two levels make the personas more specific. An instructor without stories adds little.

## Question 2
Should `personas.md` include an anti-persona: someone Pi is deliberately not built for?

*Why we ask:* Several requirements say what Pi refuses to do: it never writes code, never blocks Claude Code, and doesn't let the student type free questions (FR-0.3, FR-1.6, FR-2.5). An anti-persona, such as "an experienced developer who wants Pi to fix their code" or "a student who wants Pi to do the assignment", explains those limits in one place, and gives stories a reason for their "won't" acceptance criteria.

A) Yes, one anti-persona: a student who wants Pi to write or fix the code for them

B) Yes, one anti-persona: an experienced developer who already understands the code and finds Pi's interruptions a cost

C) Yes, both anti-personas in A and B

D) No anti-persona. The out-of-scope list in requirements §8 is enough

E) Other (please describe after [Answer]: tag below)

[Answer]: C. Both anti-personas. They sharpen what Pi must not do (write code) and what it must avoid (needless interruptions).

## Question 3
How much detail should each persona have?

*Why we ask:* The grading rubric asks for "specific" personas. A persona is only useful in design if it settles real questions, such as how long a student will leave a card open before skipping it, or whether they know what a closure is. Generic fields ("goals", "frustrations") don't settle those.

A) Short: name, one-paragraph background, three goals, three frustrations

B) Working profile: A, plus how they use Claude Code today, what they already know and don't know (named concepts from the demo app), how they react to interruptions, and what a good session looks like to them

C) Working profile as in B, plus a short "day in the life" scenario of one coding session with Pi

D) Other (please describe after [Answer]: tag below)

[Answer]: C. Working profile plus a day-in-the-life scenario, so the personas feel real.

## Part B: Breakdown and size

## Question 4
How should the stories be organized? See the "Story breakdown approaches" table above.

*Why we ask:* The grouping decides how stories line up with units later, and how easy it is to see gaps.

A) User journey: groups follow a student's session from setup to summary

B) Feature: groups follow the FR areas in requirements.md

C) Domain: groups by area (teaching, grading, before-and-after check, setup and pairing, video, cost and safety)

D) Epic-based hybrid: epics follow the user journey, and the stories inside each epic are cut by feature

E) Epic-based hybrid: epics follow domains, and the stories inside each epic are cut by user journey

F) Other (please describe after [Answer]: tag below)

[Answer]: D. Epics follow the student's journey, stories inside are cut by feature. Easy to read and maps cleanly onto units.

## Question 5
How large should each story be?

*Why we ask:* Requirements.md has about 50 numbered requirements. One story per requirement gives many small stories that are easy to test but hard to read as a whole. Merging them gives fewer, larger stories that may not fit inside one unit, or one person's work in a two-week window.

A) Small: about one story per numbered requirement (roughly 40 to 50 stories)

B) Medium: one story per thing the student or teammate can notice (roughly 25 to 35 stories)

C) Large: one story per major capability (roughly 12 to 18 stories), with more acceptance criteria each

D) Other (please describe after [Answer]: tag below)

[Answer]: B. Medium, 25 to 35 stories. Realistic for a two-week build.

## Question 6
How should the nine lesson component types (FR-2.7) appear in the stories?

*Why we ask:* Each type has its own renderer and response, and some have graders. But the component system's whole point (FR-2.6) is that the agent loop, feed and grading flow don't change per type. Stories can reflect either side of that.

A) One story per component type, nine in all, each with its own acceptance criteria

B) One story for the shared component system (schema check, feed, regenerate-then-drop), plus one story per type that the student responds to (multiple choice, written response, quick-time event, prediction). Display-only types (explanation, diagram, data graphic, video, summary) are covered as acceptance criteria inside the stories that use them

C) One story for the shared component system, with each type listed as an acceptance criterion inside it

D) Other (please describe after [Answer]: tag below)

[Answer]: B. One story for the shared component system, plus one per type the student responds to.

## Question 7
Should stories be labeled with a relative size?

*Why we ask:* INVEST asks for stories that can be estimated. A size label (S, M or L) helps Units Generation balance units across four people and two weeks. It isn't a schedule, and it can be rough.

A) Yes: S, M or L on every story, with a one-line note on what makes an L large

B) No size labels. Units Generation handles balancing

C) Other (please describe after [Answer]: tag below)

[Answer]: A. S/M/L size labels, to help Units Generation fit the two-week window.

## Part C: Format and acceptance criteria

## Question 8
What format should the acceptance criteria use?

*Why we ask:* Given/When/Then reads well for flows with a trigger, such as a wrong answer leading to a drill-down. A checklist reads better for limits and rules, such as the caps or the read-only tools. Mixing them is common, but needs a rule for when to use which.

A) Given/When/Then for every criterion

B) Checklist bullets for every criterion

C) Given/When/Then for behavior the student triggers, and checklist bullets for rules and limits

D) Other (please describe after [Answer]: tag below)

[Answer]: C. Given/When/Then for student behavior, checklists for rules and limits.

## Question 9
Pi's output comes from a model, so the same event can produce different lessons. How should acceptance criteria handle that?

*Why we ask:* A criterion like "Pi shows a relevant explanation" can't pass or fail cleanly. Some criteria can be checked by a test (the component is schema-valid, it appears within 15 seconds, the cap holds). Others need a person to judge quality, and AC-4 already sets one such check: teammates hand-review 20 exercises for correct answer keys.

A) Criteria check only what a test can decide. Teaching quality is left to AC-4's hand review

B) Every story can have both kinds, and each criterion is labeled **[test]** (automated) or **[review]** (a teammate checks it against a short rubric)

C) Same as B, but **[review]** criteria are allowed only on stories about lesson content and grading

D) Other (please describe after [Answer]: tag below)

[Answer]: C. [review] criteria only for lesson content and grading, because only teaching quality needs a human check.

## Question 10
How should failure paths appear?

*Why we ask:* Many requirements are about things going wrong: the API key is missing, a model call fails, the Builder times out and the question is dropped, a cap is reached, a video render fails, Pi is closed while Claude Code runs (FR-1.7, FR-4.5, FR-7.6, NFR-1, NFR-3). These are easy to leave out of happy-path stories.

A) Every story has at least one failure-path criterion, where a failure is possible

B) Failure-path criteria only where requirements.md names the failure

C) Separate "when things go wrong" stories, grouped together, apart from the happy-path stories

D) Other (please describe after [Answer]: tag below)

[Answer]: A. At least one failure-path criterion per story where failure is possible (bad model output, Builder not answering).

## Question 11
How should each story link back to requirements.md?

*Why we ask:* Traceability lets graders and Units Generation check that nothing is missing. It also adds clutter to every story.

A) Each story lists the FR, NFR and AC IDs it covers, and `stories.md` ends with a table from every requirement ID to its stories

B) Each story lists its IDs. No reverse table

C) Only the reverse table at the end of `stories.md`

D) Other (please describe after [Answer]: tag below)

[Answer]: A. IDs on each story plus a reverse table, so requirements and stories stay traceable.

## Part D: Parts of Pi the student doesn't see

## Question 12
Some requirements describe behavior the student never sees directly: the agent's read-only tools and step limit (FR-0), schema checks (NFR-5.6), logging (NFR-4), the hook server token (NFR-5.3), and the single config module (NFR-7.2). How should stories cover these?

*Why we ask:* Writing "As a student, I want a per-session hook token" sounds false, because a student would never ask for that. Leaving them out loses traceability for real work that takes up build time.

A) Rewrite each as a student-facing outcome where one exists ("As a student, I want Pi to never touch my repo, so that I can trust it beside my real project"), and attach the technical rule as an acceptance criterion

B) Write them as team stories in their own section ("As a Pi developer, I want every prompt in one config module, so that…")

C) A, and for the few with no honest student-facing outcome (logging, config module), team stories as in B

D) Don't write stories for these. Leave them in requirements.md, and list them in the traceability table as "covered by NFRs, no story"

E) Other (please describe after [Answer]: tag below)

[Answer]: C. Student-facing outcomes where they exist, team stories only where there is none.

## Question 13
How should Pi's agent (FR-0) be treated in stories: deciding what to teach, picking a component, asking the Builder?

*Why we ask:* FR-0 makes Pi an agent that decides on its own. Stories usually have a human actor. The agent's choices still need testable criteria, such as "ends the run after showing one component" and "shows nothing when no concept on the list applies".

A) The student stays the actor. Agent behavior becomes acceptance criteria on the student stories it affects

B) Write a small set of stories with Pi as the actor ("As Pi, I want to…"), each with criteria for its decisions

C) Other (please describe after [Answer]: tag below)

[Answer]: B. A small set of "As Pi..." stories. Pi deciding when to intervene and when to ask the Builder is the agentic core of the product and needs its own testable criteria.

## Part E: Journeys and scope

## Question 14
Should `stories.md` include the demo scenario as a worked journey?

*Why we ask:* Founding idea §7 walks through a WebSocket session step by step, but parts of it are now out of scope: Pi asking the Builder for an explanation (now a stretch goal), a pseudocode design exercise (no pseudocode input in the MVP), and scheduling a question for a future session (no memory across sessions). A journey rewritten for the MVP would show how the stories connect, and give the live demo (AC-2, AC-3) a script to follow.

A) Yes: a short journey at the top of `stories.md`, rewritten to match the MVP, with each step linking to its stories and a note on what changed from §7

B) Yes, as in A, plus a second journey for the demo teammate setting Pi up on a fresh clone without the video runner (AC-5, AC-6)

C) No journey. The stories are enough

D) Other (please describe after [Answer]: tag below)

[Answer]: B. Add the second journey: setup on a fresh clone without the video runner. That is our biggest demo risk.

## Question 15
How should the optional video feature (FR-7) be covered?

*Why we ask:* Video is real but off by default, and the demo must work without it (AC-5). It has its own checks, consent, render queue, cap and fallback, so it's not small. It's also the piece most likely to slip in a two-week build.

A) Full stories like any other feature, in their own group, each tagged **Optional feature**

B) One story covering the whole video feature, with all its criteria inside

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Video as full stories tagged "Optional feature".

## Question 16
How should stretch goals and out-of-scope items (requirements §8) appear?

*Why we ask:* Writing stretch goals as stories makes it easy to pick one up if time allows, but it can make the scope look bigger than it is. Leaving them out keeps `stories.md` to what will be built.

A) Stretch goals as short stories in a final "Stretch" section, clearly marked as not in the MVP, with no acceptance criteria. Out-of-scope items are not written as stories

B) No stretch stories. A short "Not in this release" list at the end of `stories.md` links to requirements §8

C) Stretch goals written as full stories with acceptance criteria, in a final "Stretch" section

D) Other (please describe after [Answer]: tag below)

[Answer]: B. No stretch stories, just a "Not in this release" list linking to requirements section 8. Keeps the focus tight.

---

# Generation rules (from the answers, 2026-11-24)

These rules come from Questions 1 to 16 and the four follow-ups in `story-generation-clarification-questions.md`, where the team picked option A for all four.

| Topic | Rule | Source |
|---|---|---|
| Personas | An intro-level student, an intermediate student and the demo teammate, each as a working profile with a day-in-the-life scenario | Q1 B, Q3 C |
| Anti-personas | A student who wants Pi to write or fix their code, and an experienced developer who sees Pi's interruptions as a cost | Q2 C |
| Intro student | Served only by features already in scope: first-use term definitions, explain first after a miss (FR-3.2), drill-down (FR-4.3) and skip (FR-3.5). No story changes difficulty per student (that is v2, §8) | Q1, follow-up 1 A |
| Organization | Epics follow the student's journey. The stories inside each epic are cut by feature | Q4 D |
| Size | Medium stories, 25 to 35 in the core set. Video stories don't count toward that range. Merge stories only if the core set goes over 35. Every story is labeled S, M or L, with a one-line note on what makes an L large | Q5 B, Q7 A, follow-up 4 A |
| Component types | One story for the shared component system, plus one each for multiple choice, written response, quick-time event and prediction. Display-only types appear as criteria in the stories that use them | Q6 B |
| Acceptance criteria | Given/When/Then for behavior the student triggers. Checklist bullets for rules and limits | Q8 C |
| [test] and [review] | Each criterion is labeled. [review] is allowed only on lesson-content and grading stories, including the relevance of the "As Pi…" decisions. Limits on the agent stay [test] | Q9 C, follow-up 3 A |
| Failure paths | Every story where something can fail has at least one failure-path criterion | Q10 A |
| Traceability | Every story lists its FR, NFR and AC IDs, and `stories.md` ends with a reverse table | Q11 A |
| Actors | "As Pi…" stories cover the agent's decisions: what to teach and when, which component to show, when to show nothing, when to ask the Builder, when to drop a question. Limits on the agent (read-only tools, repo boundary, secret files, step limit, caps) are student-facing trust and cost stories. Team stories ("As a Pi developer…") cover only logging and the config module | Q12 C, Q13 B, follow-up 2 A |
| Journeys | Two journeys at the top of `stories.md`. (1) The MVP demo session, with a note on what changed from founding idea §7. (2) A teammate setting Pi up on a fresh clone without the video runner (AC-5, AC-6) | Q14 B |
| Video | Full stories in their own group, each tagged **Optional feature** | Q15 A |
| Scope | No stretch stories. A short "Not in this release" list at the end links to requirements §8 | Q16 B |

---

# Execution Plan (Part 2: Generation)

Part 2 runs only after every question is answered, the answers are checked, and the team approves this plan. Each step is marked `[x]` as soon as it is done.

## Prepare
- [x] Re-read requirements.md and list every FR, NFR and AC ID to cover
- [x] Record the team's answers to Questions 1 to 16 as the rules for generation, in a short "Decisions" block at the top of `stories.md`

## Personas (`personas.md`)
- [x] Write the personas chosen in Question 1, at the detail level from Question 3
- [x] Write the anti-persona(s) from Question 2, if any
- [x] Check each persona against requirements §4 (level, language, setup context)
- [x] Add a persona-to-story map, filled in after the stories are written

## Stories (`stories.md`)
- [x] Set up the groups or epics chosen in Question 4
- [x] Write the journey or journeys from Question 14, if any
- [x] Write the stories for each group, sized per Question 5, with component types handled per Question 6
- [x] Write the stories for parts the student doesn't see, per Questions 12 and 13
- [x] Write the video stories per Question 15
- [x] Write the stretch and out-of-scope section per Question 16
- [x] For every story, write acceptance criteria in the format from Question 8, with **[test]** or **[review]** labels per Question 9 and failure paths per Question 10
- [x] Add size labels per Question 7, if chosen
- [x] Add requirement links and the traceability table per Question 11

## Check
- [x] Every story follows INVEST: Independent, Negotiable, Valuable, Estimable, Small, Testable. Split or merge any that don't
- [x] Every story has acceptance criteria
- [x] Every FR and AC in requirements.md maps to at least one story, or is marked as covered by an NFR with no story (if Question 12 allows that)
- [x] No story contradicts requirements.md or brings back an out-of-scope item
- [x] Every persona maps to at least one story, and every story to at least one persona
- [x] Fill in the persona-to-story map in `personas.md`

## Mandatory artifacts
- [x] Generate `stories.md` with user stories following INVEST criteria
- [x] Generate `personas.md` with user archetypes and characteristics
- [x] Ensure stories are Independent, Negotiable, Valuable, Estimable, Small, Testable
- [x] Include acceptance criteria for each story
- [x] Map personas to relevant user stories
