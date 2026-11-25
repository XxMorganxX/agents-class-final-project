# Units of Work

**Stage:** INCEPTION - Units Generation (Part 2: Generation)
**Sources:** `plans/unit-of-work-plan.md` (approved 2026-11-25: Q1-Q15, readings R1-R6, the team changes before approval), `plans/unit-of-work-clarification-questions.md` (CQ1-CQ5), components.md, component-methods.md, component-dependency.md, stories.md, execution-plan.md, README.md
**Related files:** [`unit-of-work-dependency.md`](unit-of-work-dependency.md) (dependencies, contracts, integration points), [`unit-of-work-story-map.md`](unit-of-work-story-map.md) (every story mapped to one unit)

Pi is one Electron app on the student's laptop, so every unit is a **module** inside one app, not a separately deployable service. A unit decides three things: which stories are built and accepted together, the order they are built in, and which folders a teammate owns while building them.

## Summary

| Order | Unit | In plain English | Stories | Sizes (S/M/L) |
|---|---|---|---|---|
| 1 | **U1a** Project Skeleton and Contracts | The empty project, its tooling, all settings and prompts, the shapes of every message, and the interfaces and fakes that let later units be tested without a real model | 1 | 1 / 0 / 0 |
| 2 | **U1b** Shared Core | The list of card types, the code that turns Claude Code hook events into triggers, the cost counter and the log | 3 | 2 / 0 / 1 |
| 3 | **U4** Lesson Component Types | The eight kinds of card: what the agent fills in, how answers are graded, how each card looks | 7 | 4 / 3 / 0 |
| 4 | **U3** Tutor Agent and Card Pipeline | The AI teacher: reads the code, decides what to teach, writes a card, checks tricky answer keys with Claude Code, drops anything broken or late | 6 | 2 / 4 / 0 |
| 5 | **U5** Session and Learning Record | The conductor: when cards appear, answers, skips, disputes, drill-downs, what the student has learned, End session and the summary | 8 | 5 / 3 / 0 |
| 6 | **U2** Builder Pairing and Setup | Connecting to Claude Code: hooks installed and removed with consent, events received safely, first-run checks | 6 | 4 / 2 / 0 |
| 7 | **U6** Desktop Shell and UI | The locked-down window, the bridge into it, the setup screen, feed, status line and settings | 2 | 1 / 1 / 0 |
| 8 | **U7** Video *(optional)* | Short concept videos through Project Feynman, only when the machine can make them | 3 | 1 / 1 / 1 |
| | | **Total** | **36** (33 core, 3 video) | |

**Build order:** U1a → U1b → U4 → U3 → U5 → U2 → U6 → U7. U7 is built only if every core unit is approved by **2026-12-10** (Q10). The unit names U1a and U1b keep the other unit numbers from the plan (team change 1).

## Rules for every unit

| Rule | Source |
|---|---|
| A story maps to the unit whose behavior its acceptance criteria check. Other units it needs are listed as supporting units. A criterion that can only be checked once a later unit exists is checked in that later unit's loop | Q2 A; decision rule |
| One unit at a time goes through the shared AI-DLC loop (Functional Design to Code Generation). Each loop aims for one to two days. While one unit is in the loop, teammates review and test the previous unit and prepare answers for the next | Q8 A |
| A unit is **done** when its stories' testable criteria pass with fakes, its pure functions have the property tests Functional Design names, it runs in the app through `createApp` (see the integration points in `unit-of-work-dependency.md`), and it passes its live check if it has one | Q11 C, CQ2 |
| **Exception for U3:** prompt fixes for AC-4 get two working days after U3's code is generated. If AC-4 is still below 90%, the score is recorded in `audit.md`, the workflow moves on to U5, and AC-4 is run again in Build and Test after more prompt fixes made as U3 changes. AC-4 itself doesn't change | Team change 2 |
| Live checks (real Haiku, `claude -p`, `claude auth status`, `make-video`) run in the opt-in live suite, never in the normal unit tests | R1, R2, NFR-8.2 |
| If a unit needs code from a unit built after it, pure shared translators and contracts move to U1, and live checks run in the loop of the unit that builds the real dependency | Team decision rule (round 2) |
| Nothing writes to the student's repo | Dependency rule 7 |
| If U7 is dropped, video-only criteria of core stories are recorded as not applicable, and the stories close without them | Team change, 2026-11-25 |

---

## U1a Project Skeleton and Contracts

- **Purpose:** give every later unit a working project and stable contracts to build against.
- **Responsibilities:**
  - Project setup: Electron Forge with Vite, one TypeScript project per process, the lint import rules that keep main and renderer code apart (dependency rule 2), Vitest and fast-check.
  - **C1 Config:** every prompt, cap, timeout, limit and the concept list, validated at start (US-33). Every later prompt change edits this file.
  - **C2 Shared Schemas:** hook payloads, the runtime file, IPC messages and events, model output, and common types (`CardId`, `ConceptId`, `CodeSnapshot`, `Grade`, `BuilderEvent`, `Trigger`, `CardView`, `CardPatch`, `StatusView`), with the `check` helper.
  - **Contracts (interfaces only):** `TutorModel` and `BuilderBridge` (CQ1), the `RecordSummaryForAgent` type (M7), and `FileSystem`, `SpawnFn` and `Clock` (M3).
  - **Fakes:** `FakeTutorModel`, `FakeBuilderBridge`, an in-memory file system, recorded spawns and a fake clock, in `test/fakes/`.
  - **C22 Composition Root skeleton:** `createApp(overrides)`, `App.start()`, `App.stop()` and `AppOverrides` without `videoRunner` (M5, CQ1).
- **Stories:** US-33.
- **Live check:** none.
- **Functional Design:** expected to be skipped (almost no business logic, as the execution plan allows). Decided when U1a starts.
- **Property-test targets for Functional Design or Code Generation:** schema round-trips for the C2 schemas.
- **Skills needed:** TypeScript project setup, Electron Forge and Vite, ESLint import rules, TypeBox, Vitest and fast-check.

## U1b Shared Core

- **Purpose:** the shared pieces that run at the center of every later unit.
- **Responsibilities:**
  - **C3 Component Registry:** register types, validate fields and responses, and build the `show_component` union (main half), plus the `CardRegistry` (renderer half).
  - **C8 Builder Events:** `toBuilderEvent`, `mergeActivity`, `readTurnDetails`, and the repo filter (CQ5).
  - **Recorded hook fixtures** in `test/fixtures/hooks/` (M4). Recording them needs a temporary hand-made hook entry, because the installer comes in U2. The entry is removed after recording.
  - **C16 Budget Ledger:** reserve and settle accounting for the three caps, plus the cap-reached announcement.
  - **C17 Event Log:** metadata JSONL in app-data, with content only in debug mode.
- **Stories:** US-13, US-27, US-32.
- **Live check:** none.
- **Property-test targets:** registry validation and schema round-trips (C3), the C8 translator and merge, and cap accounting (C16).
- **Skills needed:** TypeBox schema unions, property-based testing with fast-check, Claude Code hooks (to record the fixtures).

## U4 Lesson Component Types

- **Purpose:** the eight core card types, each built and tested as a whole (Q3 A).
- **Responsibilities:** for explanation, multiple-choice, written-response, quick-time, system-design-graph, data-graphic, prediction and session-summary: `schema.ts` (fields and response), `grader.ts` (types the student answers) and `Card.tsx` (plain-text rendering and the card's own controls, through `CardProps`).
- **Stories:** US-14, US-15, US-18, US-19, US-20, US-21, US-23.
- **Live check:** the written-response grader (and prediction judging, if Functional Design gives it a model call) is checked against real Haiku **in U3's loop**, where the real `PiTutorModel` is first built. A failure there reopens U4 (CQ4, M1).
- **Nearest-entry check when U4 finishes:** every type registers in both halves of the registry and passes its schema and grader tests with `FakeTutorModel` (CQ2).
- **Property-test targets:** field and response schema round-trips, component validation, right/wrong grading.
- **Skills needed:** React and Testing Library, TypeBox, Mermaid rendering with a fallback, small charts from inline data, grading logic.

## U3 Tutor Agent and Card Pipeline

- **Purpose:** the teaching agent and everything between a proposed component and a finished card.
- **Responsibilities:**
  - **C11 Tutor Agent:** request building, tool registration, run limits and run logging.
  - **C12 TutorModel:** `PiTutorModel` (pi `ModelRuntime`, Haiku 5.5) and the `BudgetedTutorModel` decorator. The interface is U1a's.
  - **C13 Repo Read Tools** with the path guard.
  - **C14 Builder Bridge:** `ClaudeCliBuilderBridge` with receipts. The interface is U1a's.
  - **C10 Card Pipeline:** concept check, schema check with one regeneration, snapshot, answer-key check, latency limit, and `markCodeChanged`.
- **Stories:** US-09, US-11, US-12, US-17, US-24, US-28.
- **Live check:**
  - the full **AC-4 review**: 20 exercises generated from the recorded hook fixtures, at least 90% with a correct key (CQ3)
  - U4's grader against real Haiku (CQ4)
  - `claude -p` verdicts
  - AC-4 prompt fixes have the two-working-day limit above
- **Nearest-entry check when U3 finishes:** a recorded hook fixture goes through C8 into the Tutor Agent and the Card Pipeline, with U4's types registered (CQ2, CQ5).
- **Property-test targets:** `guardPath`, `receiptCovers` and `hashAnswerKey`, component validation in the pipeline.
- **Skills needed:** LLM agents on the pi harness, prompt writing, the Claude Code CLI, file-system path security, hand-reviewing answer keys.

## U5 Session and Learning Record

- **Purpose:** coordinate a session and keep a fair record of it. Kept as one unit (Q13 A).
- **Responsibilities:**
  - **C9 Session Orchestrator:** trigger intake, the run queue with merging, cadence (`cadence.ts`), deterministic follow-ups and recovery (`recovery.ts`), caps, status, End session, the `FeedSink` interface.
  - **C15 Learning Record:** the event store and its pure rules (labels, next-lesson hints, prediction pairs, metrics, summary, `summarizeForAgent`).
- **Stories:** US-08, US-16, US-22, US-25, US-26, US-29, US-30, US-31.
- **Live check:** none. U5 makes no real external call itself.
- **App check:** U5 runs the **full-app check for U4 and U3**. Recorded fixtures go through C8, C9, C11 and C10 back to C9, with a recording `FeedSink`. A failure reopens whichever unit caused it (CQ2 B).
- **Property-test targets:** mastery labels (US-30), next-lesson hints, prediction pairs, metrics, cadence and recovery steps.
- **Skills needed:** async queues and concurrency, event sourcing, property-based testing of rules, product judgement for cadence and recovery.

## U2 Builder Pairing and Setup

- **Purpose:** connect Pi to Claude Code safely, and run the first-run checks.
- **Responsibilities:**
  - **C5 Hook Server:** `127.0.0.1` only, token and schema checks, `202` at once.
  - **C6 Hook Installer:** preview, consented merge, byte-exact backup, removal, the `pi-hook.mjs` script, and the `0600` runtime file with a new token every launch.
  - **C7 Setup Checks:** `.env` (`src/main/env.ts`), repo choice, Claude Code login, the API key check.
- **Stories:** US-01, US-03, US-04, US-05, US-06, US-07.
- **Live check:** the API key check (one real Haiku call through U3's `PiTutorModel`) and `claude auth status` (R1).
- **App check:** the real hook script posts a recorded payload to C5, and it reaches C9 through C8.
- **Property-test targets:** `mergePiHooks` and `removePiHooks`, including the install-then-remove round trip (US-04, US-05).
- **Skills needed:** Fastify, Node file permissions, JSON merging, Claude Code settings and hooks. **Needs a Mac:** real `~/.claude/settings.json`, `0600` files, `claude auth status`.

## U6 Desktop Shell and UI

- **Purpose:** the window the student sees, locked down.
- **Responsibilities:**
  - **C18 Window Shell and IPC Gateway:** hardened window, IPC handlers with schema checks, push events, `IpcGateway` as the real `FeedSink`.
  - **C19 Preload API:** `window.pi` and nothing else.
  - **C20 Renderer App:** setup screen, feed, status line, settings (the video toggle shows only when the video plug-in is in the build, reading R7), End session button.
- **Stories:** US-02, US-10.
- **Also closes later checks for earlier stories:** the feed and extensibility criteria of US-13, the window parts of US-16, US-17, US-23, US-28, US-29, US-31 and US-32, and the window side of US-01, US-03, US-04 and US-05. The full list is in the story map.
- **Live check:** none.
- **App check:** the app runs with the real window, with the model and the Builder faked.
- **Skills needed:** Electron security (sandbox, CSP, `contextBridge`), React, Testing Library.

## U7 Video (optional)

- **Purpose:** optional concept videos. A leaf: nothing in the core depends on it (D14).
- **Responsibilities:** **C21 Video Plug-in**: requirement checks, registering `offer_video`, the video component types and `pi-video://`, the render queue, fallback to text. It also owns the `VideoRunner` contract, `MakeVideoRunner`, `FakeVideoRunner` and the `videoRunner` field in `AppOverrides` (CQ1).
- **Stories:** US-V1, US-V2, US-V3.
- **Live check:** `make-video` (R1).
- **Built only if** every core unit is approved by 2026-12-10. If it is dropped, no other unit changes (CQ1). The video-only criteria of four core stories (US-07, US-09, US-27, US-28) are then recorded as not applicable, and those stories close without them (story map, "Rule for a dropped U7"). The same applies to AC-5's "turning videos on shows what is missing" and J2 step 5 (Clarification Question 6 A). The first half of AC-5 is always required.
- **Skills needed:** Project Feynman, Python and `uv`, background subprocesses. **Needs a Mac with a Metal GPU** (FR-7.2).

---

## Team roles and ownership

Owners and reviewers are assigned at the first Construction meeting (Q7 B). The README's five workstreams guide the assignment (Q1). This file records the constraints and roles to assign:

| Role | What it covers |
|---|---|
| Unit owner (one per unit) | Answers the unit's Functional Design and NFR questions, reviews its generated code, runs its checks |
| Unit reviewer (one per unit) | Reviews the code and runs the real-app and live checks on a Mac where the unit has them |
| **Testing and demo** (Q9) | The Playwright Electron end-to-end test in `test/e2e/`, the AC-1 to AC-6 checks and journeys J1 and J2 in Build and Test, preparing the live demo on the WebSocket chat app built live with Claude Code (AC-1, AC-2) |

**Constraints for the assignment (Q7, R3):**
- Pi targets macOS (NFR-6.1). One teammate, Alex, doesn't have a Mac.
- Alex doesn't own U7, which needs a Mac with a Metal GPU.
- Any unit Alex owns gets a reviewer with a Mac who runs its real-app and live checks.
- Units tested mostly with fakes suit him best: U1a, U1b, U4 and U5.
- U2, U3 and the testing and demo role need a Mac for their live work.

## Code organization strategy (greenfield)

**Layout:** the approved by-process layout from Application Design (D1, components.md "Source layout") stays as it is (Q14 A). It is not the `src/{unit-name}/` pattern from `code-generation.md`. Each unit owns folders, and in four places it owns single files inside another unit's folder.

| Unit | Owns |
|---|---|
| U1a | Repo-root build files (`package.json`, Forge, Vite, TypeScript, ESLint and Vitest config); `src/shared/config.ts`; `src/shared/schemas/`; `src/main/app.ts` (the skeleton; see "Shared edit point" below); `src/main/ports.ts` (`FileSystem`, `SpawnFn`, `Clock`); `src/main/tutor/tutor-model.ts` (interface only); `src/main/builder-bridge/builder-bridge.ts` (interface only); `src/main/learning-record/agent-summary.ts` (`RecordSummaryForAgent` type only); `test/fakes/` except `FakeVideoRunner` |
| U1b | `src/components/registry/`; `src/main/builder-events/`; `src/main/budget/`; `src/main/event-log/`; `test/fixtures/hooks/` |
| U4 | `src/components/explanation/`, `multiple-choice/`, `written-response/`, `quick-time/`, `system-design-graph/`, `data-graphic/`, `prediction/`, `session-summary/` |
| U3 | `src/main/tutor/` and `src/main/builder-bridge/` (except U1a's interface files); `src/main/repo-tools/`; the card pipeline files in `src/main/session/` (`card-pipeline.ts` and its tests) |
| U5 | `src/main/session/` (except U3's card pipeline files); `src/main/learning-record/` (except U1a's `agent-summary.ts`) |
| U2 | `src/main/hook-server/`; `src/main/hook-installer/`; `src/main/setup/`; `src/main/env.ts`; `resources/pi-hook.mjs` |
| U6 | `src/main/window/`; `src/main/ipc/`; `src/preload/`; `src/renderer/` |
| U7 | `src/plugins/video/`; `test/fakes/` `FakeVideoRunner`; the `videoRunner` field in `AppOverrides` |
| Build and Test | `test/e2e/` |

**File-level ownership.** Four places hold one unit's files inside another unit's folder. Each one keeps an approved component in its approved folder (no design change):
- `tutor-model.ts`, `builder-bridge.ts` (CQ1)
- `agent-summary.ts` (M7)
- the card pipeline in `session/` (components.md puts C9 and C10 in one folder)

`src/main/ports.ts` is a new file in `src/main/`, next to `app.ts` and `env.ts`, not a new folder.

**Shared edit point (M5).** `src/main/app.ts` belongs to U1a. Each later unit adds its own wiring to it when it joins the app (Q6 A). That edit is part of the joining unit's Code Generation and is reviewed with it.

**Tests (Q15 A):**
- Unit and property tests sit next to the code they test (`foo.test.ts` beside `foo.ts`).
- Shared fakes go in `test/fakes/` and recorded hook payloads in `test/fixtures/hooks/`.
- The end-to-end test goes in `test/e2e/`.
- The opt-in live suite is tagged so it never runs with the normal tests (NFR-8.2).

## Schedule risk (R5, updated)

Recorded as a risk, not a decision. The team confirms the Construction start date at its first Construction meeting.

| | Working days |
|---|---|
| Seven core loops at one to two days each | 7 to 14 |
| Plus U3's AC-4 time limit | up to 16 |
| Weekdays from 2026-11-25 to the 2026-12-10 video cutoff | 12 (10 without Thanksgiving, 2026-11-26 and 27) |
| Weekdays from 2026-11-25 to the 2026-12-16 final deadline | 16 (14 without Thanksgiving) |

- To reach the video cutoff, the loops have to average about one and a half days, starting right after this milestone.
- Even the best case (7 days) needs Construction to start by 2026-12-02.
- In the worst case, every weekday up to 2026-12-16 goes to unit loops, leaving no time for Build and Test, the demo video or the presentation.
- U1a's loop is shorter if it skips Functional Design.

## Validation

- **Every story in exactly one unit:** 36 of 36 (33 core and 3 video). See `unit-of-work-story-map.md`.
- **Every component in exactly one unit:** C1 to C22 are each assigned. C22 belongs to U1a, and later units add wiring to it (M5).
- **Build order has no backward dependency:** each unit depends only on units before it (`unit-of-work-dependency.md`).
- **Video is a leaf:** no unit depends on U7, and dropping it changes no other unit's code. Core stories, AC-5's second half and J2 step 5 close their video-only criteria as not applicable (story map, CQ6 A).
- **Each contract has one owner:** see the contract table in `unit-of-work-dependency.md`.
- **Design files unchanged:** these units decide which unit builds a file, not what it contains.
