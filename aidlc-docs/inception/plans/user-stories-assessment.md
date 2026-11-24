# User Stories Assessment

**Stage:** INCEPTION - User Stories (Part 1, Step 1)
**Date:** 2026-11-23
**Inputs:** `inception/requirements/requirements.md` (approved 2026-11-23), `docs/founding-idea.md`, `aidlc-state.md` (Run Scope)

## Request analysis

- **Original request:** Build Pi, a teaching assistant agent that runs as an Electron sidecar beside Claude Code. It explains the code Claude writes and checks that the student understands it, without writing to the student's repo.
- **User impact:** Direct. Pi is a new product, and almost every requirement describes something the student sees or does: the setup screen, the lesson feed, exercises, drill-downs, the before-and-after check and the session summary.
- **Complexity level:** Complex. The requirements span nine functional areas (FR-0 to FR-8), eight non-functional areas, nine lesson component types and six acceptance criteria.
- **Stakeholders:**
  - the student using Pi (primary user, requirements §4)
  - the teammate who installs and runs the live demo (secondary user, §4)
  - the four-person project team, who build it in a two-week Construction window
  - the course graders, who assess the stories and personas under criterion D

## Assessment criteria met

- [x] **High priority: new user features.** The whole product is new and user-facing.
- [x] **High priority: user experience changes.** Pi adds a second window and a teaching loop to an ordinary Claude Code session. How often it interrupts (FR-3.3, FR-6.5 interruption cost) is a core design concern.
- [x] **High priority: more than one persona.** A student at the intro-to-intermediate range, and a teammate who sets Pi up on a fresh machine with or without the video runner.
- [x] **High priority: complex business logic.** Grading verdicts, drill-down paths, mastery labels ("solid" needs two exercise types), answer-key holds, and per-session caps all have several scenarios.
- [x] **High priority: cross-team work.** Four teammates must share one picture of the product before Application Design splits it into units.
- [x] **Medium priority: testing.** AC-3 to AC-6 need user-level acceptance checks, and NFR-8 calls for property-based tests that stories can point to.
- [x] **Medium priority: options.** Several requirements allow more than one valid build, for example how the "after" prediction gets its actual result (FR-5.3).

## Decision

**Execute User Stories:** Yes.

**Reasoning:** The team's Run Scope already requires this stage. It also earns its place on the merits. Requirements.md is written from the system's side ("Pi does X"). Stories rewrite it from the student's side, which exposes gaps that a requirement list hides. For example: what the student sees when a flagged question is dropped, or how a skip feels compared with a wrong answer. Application Design and Units Generation will both map back to these stories, so they need to exist and be testable first.

## Expected outcomes

- Every FR and AC in requirements.md traces to at least one story, and every story traces back to a requirement.
- Each story has acceptance criteria that the team can check by hand or in a test, including what happens when things fail.
- Personas specific enough to settle wording and pacing questions in design: how much a term needs defining, and how many interruptions a student will tolerate.
- A story set sized and grouped so that Units Generation can map every story to one unit within the two-week window.
