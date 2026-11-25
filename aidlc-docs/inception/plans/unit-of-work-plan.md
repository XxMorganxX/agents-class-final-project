# Unit of Work Plan

**Stage:** INCEPTION - Units Generation (Part 1: Planning)
**Sources:** `application-design/` (approved 2026-11-25: components.md, component-methods.md, services.md, component-dependency.md, application-design.md), stories.md (36 stories), requirements.md, execution-plan.md, README.md (team and collaboration plan), Run Scope in `aidlc-state.md`, `construction/code-generation.md` (structure patterns)

## Scope

**Already decided, not reopened here:**
- Pi is one Electron app on the student's laptop. It deploys as one app, so every unit is a **module** inside one app, not a separately deployable service.
- The 22 components (C1-C22), the six services (S1-S6), the source layout (D1) and the four NFR-7.1 contracts.
- Video is optional and a leaf (D14). Construction must fit the two-week build window (Run Scope).
- Every story must map to **exactly one** unit before Construction (execution-plan quality gate).

**What this stage decides:** how the components and stories group into units, the order the units are built in, who owns them, and how the units map onto the code layout.

## Draft decomposition (for discussion)

This draft is a starting point for the questions below. Your answers can change it. **Replaced by "Final decomposition (for approval)" below, which reflects the answers and clarifications.**

| Draft unit | Components | Draft stories | Count |
|---|---|---|---|
| U1 Foundation and Contracts | C1 Config, C2 Shared Schemas, C3 Component Registry, C16 Budget Ledger, C17 Event Log, C22 Composition Root, test fakes | US-32, US-33 | 2 |
| U2 Builder Pairing and Setup | C5 Hook Server, C6 Hook Installer (and `pi-hook.mjs`), C7 Setup Checks, C8 Builder Events | US-01, US-03, US-04, US-05, US-06, US-07 | 6 |
| U3 Tutor Agent and Card Pipeline | C10 Card Pipeline, C11 Tutor Agent, C12 TutorModel, C13 Repo Read Tools, C14 Builder Bridge | US-09, US-11, US-12, US-17, US-24, US-27, US-28 | 7 |
| U4 Lesson Component Types | C4 (eight types: schema, grader, card) | US-14, US-15, US-18, US-19, US-20, US-21, US-23 | 7 |
| U5 Session and Learning Record | C9 Session Orchestrator, C15 Learning Record | US-08, US-16, US-22, US-25, US-26, US-29, US-30, US-31 | 8 |
| U6 Desktop Shell and UI | C18 Window Shell and IPC Gateway, C19 Preload API, C20 Renderer App | US-02, US-10, US-13 | 3 |
| U7 Video (optional) | C21 Video Plug-in | US-V1, US-V2, US-V3 | 3 |
| | | **Total** | **36** |

Notes on the draft:
- Many stories touch components in more than one unit (see "Story coverage" in components.md). The draft puts each story in the unit that holds the behavior its acceptance criteria check. Question 2 asks whether that rule is right.
- U1 has few stories because it is mostly contracts that other units build on.
- The README's Construction plan names five workstreams: session integration, question generation, answer evaluation and learning records, the user interface, and testing the demo. Question 1 option B is a decomposition that follows those workstreams instead.

## Questions

Please answer each question by writing the letter after its `[Answer]:` tag. If no option fits, choose the last option (Other) and describe what you want after the tag. You can add a short reason after any letter.

### Part A: Story grouping

## Question 1
How should the system be split into units?

A) The seven draft units above (U1-U7): Foundation, Builder Pairing and Setup, Tutor Agent and Card Pipeline, Lesson Component Types, Session and Learning Record, Desktop Shell and UI, Video

B) Follow the README's five workstreams: Session Integration (C5-C8, C9, C22), Question Generation (C10-C14, C3), Answer Evaluation and Learning Record (C4 graders, C15, C16), User Interface (C18-C20, C4 cards), Demo and End-to-End Testing (Playwright test, demo app, AC checks), plus Video as a sixth, optional unit. Foundation pieces (C1, C2, C17) would go to whichever unit needs them first

C) Fewer, larger units: Main-process Core (C1-C17, C22), Window (C18-C20), Video (C21)

D) Thin vertical slices: Slice 1 is one explanation card end to end (hook to card on screen). Each later slice adds a feature across all layers (exercises and grading, recovery, End session and summary, setup and hooks, video)

E) Other (please describe after [Answer]: tag below)

[Answer]: A. The seven draft units follow Alex's approved components, so each unit owns clear folders. Seven small units of one to two days each fit the two-week window under Q8. The README's five workstreams will guide who owns which unit when we assign people (Q7).

## Question 2
A story often needs components from several units. Which unit should it map to in `unit-of-work-story-map.md`?

A) The unit that holds the behavior its acceptance criteria check. The map also lists the other units it needs as "supporting units"

B) The unit that finishes the story last, which is usually the one that wires it in (often the session or UI unit)

C) Split such stories into one sub-story per unit, so each piece maps to exactly one unit

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Map each story to the unit whose behavior its acceptance criteria check, and list the supporting units. This keeps "exactly one unit per story" honest.

## Question 3
Each lesson component type has `schema.ts`, `grader.ts` (main) and `Card.tsx` (renderer) in one folder (D1). Where should the eight core types go?

A) One unit owns all three parts of every type, as in draft U4, so a type is built and tested as a whole

B) Split by process: `schema.ts` and `grader.ts` go to the evaluation or session unit, `Card.tsx` goes to the UI unit

C) Spread the types out by purpose: display-only types (explanation, system-design-graph, data-graphic) with the UI unit, exercise types (multiple-choice, written-response, quick-time, prediction) and session-summary with the session or evaluation unit

D) Other (please describe after [Answer]: tag below)

[Answer]: A. One unit owns the schema, grader and card for every type, so each type is built and tested as a whole. This matches the one-folder-per-type design.

### Part B: Dependencies and integration

## Question 4
Most units depend on Config, Shared Schemas, the Component Registry, the Event Log and the test fakes. How should those shared pieces be built?

A) As their own first unit (draft U1). It is built and approved before the others start, and it keeps the contracts stable

B) No separate unit. Each unit adds the shared pieces it needs, and a second teammate reviews every change to `src/shared/` and `src/components/registry/`

C) A short "contracts first" unit with only the types, interfaces and fakes (no real logic). The real Budget Ledger, Event Log and composition root move into the units that use them

D) Other (please describe after [Answer]: tag below)

[Answer]: A, plus one addition: U1 also holds the interfaces (not the implementations) of TutorModel, Builder Bridge and Video Runner, because the shared fakes in test/fakes/ implement them. Then every later unit can be built against stable contracts in any order.

## Question 5
In what order should the units be built?

A) Contracts and foundation first. Then pairing, tutor and pipeline, lesson types, and UI, each against fakes. Then the session unit, which joins them. Then video

B) A walking skeleton first: the thinnest working path from a hook event to one explanation card on screen, crossing every unit. Then each unit fills in its own stories

C) Riskiest first: the tutor agent and card pipeline (model output quality, AC-4), then session, then the rest, with video last

D) Other (please describe after [Answer]: tag below)

[Answer]: A, with one change: after U1, build U3 (Tutor Agent and Card Pipeline) and U4 (Lesson Component Types) next, back to back, U3 first. Answer-key quality (AC-4, 90% correct) is our biggest risk, and it depends on both the agent and the card types it fills. Building them early gives us the most time to fix it. Then U2, U6, U5, and video last.

## Question 6
When should the units first run together as one app through `createApp`?

A) As soon as two units are done. After that, every new unit joins the running app (with the model and Builder faked) before it counts as done

B) Once, after every core unit is done, in Build and Test

C) At a fixed point mid-build, for example the end of week 1, then again at the end

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Join units into the running app as soon as two are done, so connection problems show up early instead of in the last days.

### Part C: Team alignment

## Question 7
How should ownership be recorded in `unit-of-work.md`?

A) Name an owner and a reviewer for each unit now, as the README's collaboration plan says. Write the assignments after the tag (for example "U2 owner Zaid, reviewer Morgan")

B) Name only the skills each unit needs now. Assign people when Construction starts

C) Pair two teammates on each unit, with no single owner

D) Other (please describe after [Answer]: tag below)

[Answer]: B. Name the skills each unit needs now. The team assigns an owner and a reviewer for each unit at the first Construction meeting, as the README's collaboration plan says. One constraint to record for those assignments: Alex doesn't have a Mac. Pi targets macOS (NFR-6.1) and video needs a Mac with a Metal GPU (FR-7.2). So Alex shouldn't own U7 Video, and any unit he owns needs a reviewer with a Mac who runs the real-app and live checks (Q11). Units that are mostly tested with fakes, like U1 Foundation, U4 Card Types and the learning-record rules in U5, suit him best.

## Question 8
AI-DLC finishes one unit (Functional Design through Code Generation) before it starts the next, inside one shared workflow. How should five teammates work around that?

A) Keep the rule. Run units through the shared workflow one at a time, in build order, and keep units small so each loop takes one or two days. While one unit is in the loop, teammates review and test the unit before it, and prepare the answers for the next one

B) Keep the rule for design stages, but let teammates write code for a unit on a branch while the next unit is being designed. Record this as a workflow change in audit.md

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Keep the one-unit-at-a-time rule. Writing code on side branches outside the workflow would leave the record incomplete. Keep each loop to one or two days. While one unit is in the loop, teammates review and test the unit before it and prepare the answers for the next one.

### Part D: Technical considerations

## Question 9
Who owns the end-to-end test (Playwright on Electron), the WebSocket chat demo app and the AC-1 to AC-6 checks?

A) A separate unit, "Integration and Demo", with its own stories (J1, J2 and the AC checks). This matches the README's "testing the demo" workstream

B) No unit. Build and Test covers them after every unit is done

C) The session unit (draft U5), since it joins everything

D) Other (please describe after [Answer]: tag below)

[Answer]: B. Build and Test covers the end-to-end test and the AC checks. The demo chat app is built live during the demo (AC-1), so it isn't code we write ahead of time. Add a "testing and demo" role to unit-of-work.md, assigned with the unit owners (Q7), so this work has an owner from the start.

## Question 10
When does the video unit get built?

A) Last, and only if every core unit is done and approved by a set date. Name the date after the tag (for example 2026-12-10)

B) In parallel from the start, by the teammate who knows Project Feynman, since it is a leaf and only uses core contracts

C) Not in this build. Keep the stories and the unit, but mark the unit "deferred"

D) Other (please describe after [Answer]: tag below)

[Answer]: A, cutoff 2026-12-10. Video is built last, and only if every core unit is approved by then. That leaves time for the demo video and presentation before Dec 16.

## Question 11
When is a unit "done"?

A) When its stories' acceptance criteria pass in tests with fakes, its pure functions have the property tests Functional Design named, and the team approves its Code Generation

B) Option A, plus it runs in the real app through `createApp` (see Question 6)

C) Option B, plus a live check against the real Haiku model or the real `claude -p` wherever the unit calls one

D) Other (please describe after [Answer]: tag below)

[Answer]: C. Tests with fakes and a run in the real app aren't enough for the tutor unit. AC-4 can only be checked against the real Haiku model, so any unit that calls Haiku or claude -p also gets a live check.

### Part E: Business domain

## Question 12
First-run setup (E1: US-01 to US-05) has a main-process part (checks, hook installer) and a window part (the setup screen). Where should setup sit?

A) With pairing (draft U2), because setup is mostly about Claude Code, the hooks and the API key. The UI unit only builds the setup screen

B) With the UI unit, because the student meets setup as a screen

C) As its own unit, "Setup", apart from pairing (hook server and events)

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Setup checks and the hook installer stay with pairing (U2). US-02 stays in U6 as in the draft, because its acceptance criteria check what the setup screen shows. U2 is its supporting unit.

## Question 13
The learning record (C15) and the session orchestrator (C9) are one unit in the draft. Should they stay together?

A) Together. The orchestrator is the only writer to the record (D6), and End session and the summary need both

B) Apart. The learning record's pure rules (labels, pairs, metrics, summary) form a unit that can be built and property-tested first, without any orchestration

C) Other (please describe after [Answer]: tag below)

[Answer]: A. The orchestrator is the only writer to the record, and End session needs both. Splitting them would add another unit loop to a tight two-week schedule.

### Part F: Code organization (greenfield, multi-unit)

## Question 14
`code-generation.md` suggests `src/{unit-name}/` for a multi-unit monolith. The approved design (D1, components.md "Source layout") splits code by process instead: `src/main/<feature>/`, `src/renderer/`, `src/components/<type>/`, and so on. Which should Construction follow?

A) Keep the approved by-process layout. `unit-of-work.md` lists the folders each unit owns. No design file changes

B) Switch to `src/{unit-name}/`, with process subfolders inside each unit. This would change the source layout in components.md and D1 in application-design.md, and the lint and tsconfig rules in component-dependency.md

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Keep Alex's approved by-process layout. unit-of-work.md lists the folders each unit owns. No design changes.

## Question 15
Where do tests go?

A) Next to the code they test (`foo.test.ts` beside `foo.ts`). Shared fakes and fixtures stay in `test/fakes/` and `test/fixtures/` as the design says, and the end-to-end test goes in `test/e2e/`

B) In `test/{unit-name}/`, mirroring each unit, as `code-generation.md` suggests. Shared fakes and fixtures stay in `test/`

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Tests go next to the code, shared fakes and fixtures stay in test/, and the end-to-end test goes in test/e2e/.

## Answer analysis (Step 4)

All 15 answers are filled in. Three issues need the team's decision. They are in `unit-of-work-clarification-questions.md`:

| # | Issue | Answers | Clarification |
|---|---|---|---|
| 1 | Contradiction: U1 owns interfaces that sit in U3's and U7's folders, but Q14 says no design changes | Q4, Q1, Q14 | Question 1 |
| 2 | Contradiction: the "runs in the real app" check comes before the orchestrator, hooks and window exist | Q5, Q6, Q11 | Question 2 |
| 3 | Gap: U3's live AC-4 check needs U4's exercise types, which are built after U3 | Q5, Q11, Q3 | Question 3 |

These readings were settled from the answers without a question. The team can correct any of them at approval:

| # | Reading | Answers |
|---|---|---|
| R1 | Which units get the Q11 live check: U2 (the API key check is one real Haiku call; `claude auth status`), U3 (agent runs on Haiku; `claude -p`), U4 (written-response checklist grading and prediction judging call Haiku through `generateJson`), U7 (`make-video`). U1, U5 and U6 make no real external call, so they have no live check. Q11 names "Haiku or claude -p", so `claude auth status` and `make-video` are a reading | Q11 |
| R2 | Live checks go in the opt-in live suite, never in the normal unit tests, which never call a real model (NFR-8.2) | Q11 |
| R3 | The Q7 notes on Alex are guidance for the owner assignments, not assignments. Ownership is per unit: U5 stays one unit (Q13), so "the learning-record rules in U5" means U5 suits him as owner or co-reviewer with a Mac reviewer for its app checks | Q7, Q13 |
| R4 | Journeys J1 and J2 and the AC-1 to AC-6 checks aren't stories, so the story map lists them under Build and Test with the "testing and demo" role as owner, not under a unit | Q9, Q2 |
| R5 | Schedule check: six core loops at one to two days each take 6 to 12 working days. To have every core unit approved by the 2026-12-10 video cutoff, Construction has to start between about 2026-11-28 and 2026-12-04. `unit-of-work.md` will record this as a schedule risk, not a decision. **Replaced by R5 (updated) in "Team changes before approval", after the U1 split and the U3 time limit** | Q8, Q10 |
| R6 | US-02 maps to U6 with U2 as its supporting unit, which follows the Q2 rule | Q12, Q2 |

**Team review (2026-11-25):** the team accepted R1 to R6. R5's start window is a schedule risk, and the start date will be confirmed at the first Construction meeting.

### Clarification outcomes (round 1)

| # | Answer | Effect on the plan |
|---|---|---|
| CQ1 | C | U1 owns `src/main/tutor/tutor-model.ts` and `src/main/builder-bridge/builder-bridge.ts` (interfaces only) and their fakes. `VideoRunner`, `FakeVideoRunner` and the `videoRunner` override all belong to U7. No design change |
| CQ2 | B, with A's check before U5 | Build order is **U1, U4, U3, U5, U2, U6, U7**. U4 is checked through the registry and U3 through a recorded fixture into the Tutor Agent. U5 then runs the full-app check for both, and a failure reopens the unit that caused it |
| CQ3 | C | U4 before U3, so U3's live check is the full AC-4 review (20 exercises, 90% correct) |

The new order opens two gaps, because U4 and U3 now need code from units built after them. They are round 2 in `unit-of-work-clarification-questions.md`:

| # | Issue | Clarification |
|---|---|---|
| 4 | Gap: U4's live grader check (R1) needs the real `PiTutorModel`, which is C12 in U3 | Question 4 |
| 5 | Gap: U3's fixture check and U5 need C8 Builder Events to turn recorded hook payloads into triggers, but C8 is in U2, built fifth | Question 5 |

### Clarification outcomes (round 2)

| # | Answer | Effect on the plan |
|---|---|---|
| CQ4 | A | U4 is done with fakes and its registry check. U3's live check also grades real written responses through U4's grader against Haiku, and a failure reopens U4. U1 still holds interfaces only |
| CQ5 | A | C8 Builder Events moves from U2 to U1. The AC-4 review runs on real recorded hook fixtures. US-03 and US-06 stay in U2, with U1 as a supporting unit |

### Decision rule and the moves made under it

The team's rule (2026-11-25): *if a unit needs code from a unit built after it, move pure shared translators and contracts to U1, and run live checks in the loop of the unit that builds the real dependency. Record each move here instead of opening another round.*

I checked every unit in build order for code it needs from a later unit:

| # | Case | Move |
|---|---|---|
| M1 | U4's live grader check needs `PiTutorModel` (U3) | CQ4 A: check runs in U3's loop |
| M2 | U3 and U5 need C8 (draft U2) | CQ5 A: C8 to U1 |
| M3 | The injected interfaces `FileSystem`, `SpawnFn` and `Clock` are used by U1's own C8 and C17 and by every later unit, and the design gives them no owner | Contracts, so U1 owns them, with their in-memory fakes in `test/fakes/` |
| M4 | U1's C8 tests and U3's AC-4 review both need the recorded hook payloads | U1 owns `test/fixtures/hooks/` and records them |
| M5 | C22 Composition Root (U1) wires components from every later unit | U1 builds the `createApp` skeleton and `AppOverrides` (without `videoRunner`, per CQ1). Each later unit adds its own wiring to `src/main/app.ts` when it joins the app (Q6). The other direction, so no code moves |

Checked, no move needed:
- `FeedSink` is defined with C9 (U5) and implemented by `IpcGateway` (U6). U6 comes after U5, and U5 tests with an array recorder.
- U2's live API-key check needs `PiTutorModel`, and U3 comes before U2.
- U6 needs C6, C7 (U2) and C9 (U5), all built earlier.
- U7 uses only core contracts.

None of M1 to M5 changes a design file. They decide which unit builds a file, not what the file contains.

Two more cases came up while mapping stories in Part 2 and were settled under the same rule:

| # | Case | Move |
|---|---|---|
| M6 | US-12's "before prediction first" and "explain first" criteria depend on `nextLessonHint` in C15 (U5), built after U3 | The criteria are checked in U5's loop. US-12 stays in U3 |
| M7 | The Tutor Agent's `AgentRunContext` (U3) holds a `RecordSummaryForAgent`, a type declared with C15 (U5) | The type is a contract, so U1a owns it in `src/main/learning-record/agent-summary.ts` (type only), the same file-level pattern as CQ1 |

Also placed in Part 2: the M3 interfaces go in `src/main/ports.ts`, a new file next to `app.ts`, not a new folder. C10's files in `src/main/session/` belong to U3, and the rest of that folder to U5, because components.md puts C9 and C10 in one folder.

## Team changes before approval (2026-11-25)

Before approving, the team asked whether any unit was too big for the one-to-two-day loop (Q8). U1 was the main concern. It held about ten pieces for two stories, and as the first unit it also has to set up the project. The team made three changes.

### Change 1: U1 split into U1a and U1b

The team chose this because smaller loops give the AI a smaller context, and the team gets an approval checkpoint before everything else builds on U1. The names U1a and U1b keep the other unit numbers the same.

| Unit | Holds | Story |
|---|---|---|
| **U1a Project Skeleton and Contracts** | Project setup (Electron Forge with Vite, one TypeScript project per process, the lint rules for process boundaries, Vitest, fast-check); C1 Config; C2 Shared Schemas; the `TutorModel` and `BuilderBridge` interfaces (CQ1); `FileSystem`, `SpawnFn` and `Clock` (M3); the fakes for all of these in `test/fakes/`; the C22 `createApp` skeleton and `AppOverrides` (M5) | US-33 |
| **U1b Shared Core** | C3 Component Registry; C8 Builder Events (CQ5) with the recorded hook fixtures in `test/fixtures/hooks/` (M4); C16 Budget Ledger; C17 Event Log | US-32 |

- U1b depends only on U1a. U4 starts after U1b, because U4 needs the registry.
- Other units' references to U1 now mean U1a or U1b. M1 to M5 and CQ1 still apply as written.
- U1a has almost no business logic, so the team expects it to skip Functional Design, which the execution plan allows ("a unit with no business logic, such as pure wiring, may skip it"). That choice is made when U1a starts.
- Recording real hook fixtures in U1b needs a temporary hand-made hook entry, because the installer comes in U2. The entry is removed after recording.

### Change 2: a time limit on U3's AC-4 prompt fixes

- Prompt fixes for AC-4 get **two working days after U3's code is generated**.
- If AC-4 is still below 90% at the end of the two days, the score is recorded in `audit.md` and the workflow moves on to U5.
- AC-4 is then run again in Build and Test, after more prompt fixes made through the workflow as changes to U3.
- **AC-4 itself doesn't change.** This is an exception to Q11 C for U3 only: U3 can be approved with a recorded AC-4 score below 90%, and the 90% target is then checked in Build and Test.
- Every prompt lives in C1 Config (NFR-7.2), and C1 is U1a's file. AC-4 prompt fixes are still made and approved as U3 work, because they change U3's behavior.

### Change 3: R5 (updated), the schedule check for eight loops

- **Core loops:** seven (U1a, U1b, U4, U3, U5, U2, U6). At one to two days each, that is 7 to 14 working days. U3's time limit can add up to 2, so the core takes **7 to 16 working days**.
- **Days available:** counting weekdays only, there are 12 from 2026-11-25 to the 2026-12-10 video cutoff, and 16 to the 2026-12-16 final deadline. Thanksgiving (2026-11-26 and 27) is counted as a working day. If the team doesn't work those two days, there are 10 days to the cutoff and 14 to the deadline.
- **Video cutoff:** to have every core unit approved by 2026-12-10, the loops have to average about one and a half days, starting right after this milestone. Even the best case (7 days, no AC-4 extension) means Construction must start by 2026-12-02.
- **Final deadline:** the worst case (16 days) uses every weekday up to 2026-12-16 and leaves no time for Build and Test, the demo video or the presentation.
- `unit-of-work.md` records this as a schedule risk, not a decision. The team confirms the start date at the first Construction meeting, as agreed for R5. If U1a skips Functional Design, its loop is shorter than the others.

## Final decomposition (for approval)

**Build order:** U1a → U1b → U4 → U3 → U5 → U2 → U6 → U7 (optional; built only if every core unit is approved by 2026-12-10).

| Order | Unit | Components | Stories (draft; final map in Step 9) | Live check (R1, CQ4) |
|---|---|---|---|---|
| 1 | U1a Project Skeleton and Contracts | Project setup; C1, C2; `TutorModel` and `BuilderBridge` interfaces; `FileSystem`, `SpawnFn`, `Clock`; fakes; C22 skeleton | US-33 | None |
| 2 | U1b Shared Core | C3, C8 with hook fixtures, C16, C17 | US-32 | None |
| 3 | U4 Lesson Component Types | C4 (eight types: schema, grader, card) | US-14, US-15, US-18, US-19, US-20, US-21, US-23 | Runs in U3's loop |
| 4 | U3 Tutor Agent and Card Pipeline | C10, C11, C12 (real and budgeted), C13, C14 (real) | US-09, US-11, US-12, US-17, US-24, US-27, US-28 | AC-4 review on real fixtures (two-day limit on prompt fixes, Change 2); U4 grader check; `claude -p` |
| 5 | U5 Session and Learning Record | C9, C15 | US-08, US-16, US-22, US-25, US-26, US-29, US-30, US-31 | None; runs the full-app check for U4 and U3 |
| 6 | U2 Builder Pairing and Setup | C5, C6 (and `pi-hook.mjs`), C7 | US-01, US-03, US-04, US-05, US-06, US-07 | API key call; `claude auth status` |
| 7 | U6 Desktop Shell and UI | C18, C19, C20 | US-02, US-10, US-13 | None |
| 8 | U7 Video (optional) | C21, `VideoRunner`, `FakeVideoRunner` | US-V1, US-V2, US-V3 | `make-video` |

Total: eight units (seven core and one optional) and 36 stories (33 core and 3 video), each in one unit. Rules that apply to every unit:
- Q2: a story maps to the unit whose behavior its acceptance criteria check, with supporting units listed.
- Q11 C: done means fakes, plus the real-app check, plus a live check where the unit has one. The one exception is U3's AC-4 time limit (Change 2).
- Q8: one unit at a time in the shared workflow.
- Q7, R3: owners and reviewers are assigned at the first Construction meeting, along with the testing and demo role (Q9). Alex doesn't own U7, and any unit he owns gets a reviewer with a Mac.
- Q14 A, Q15 A: by-process layout, tests next to the code.

## Execution checklist

### Part 1: Planning
- [x] Step 1: Read the approved application design, stories, requirements, execution plan and README
- [x] Step 2: Draft a decomposition and write the questions (this file)
- [x] Step 3: Team answers every [Answer]: tag (15 answers, 2026-11-25)
- [x] Step 4: Analyze the answers for gaps, contradictions and vague answers; add follow-up questions if needed (2 rounds, 5 clarifications answered; moves M1-M5 recorded under the team's decision rule)
- [x] Step 5: Team approves the plan (approved 2026-11-25, after the U1 split and the AC-4 time limit)

### Part 2: Generation
- [x] Step 6: Generate `aidlc-docs/inception/application-design/unit-of-work.md` with unit definitions, responsibilities, components, owners and build order
- [x] Step 7: Document the code organization strategy in `unit-of-work.md` (greenfield): folders each unit owns, and where tests go
- [x] Step 8: Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md` with the dependency matrix between units, the contracts each one uses, and the integration points
- [x] Step 9: Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md` mapping every story to exactly one unit
- [x] Step 10: Validate unit boundaries and dependencies (no cycles, video is a leaf, shared contracts have one owner)
- [x] Step 11: Check that all 36 stories are assigned (33 core and 3 video), each to exactly one unit
- [x] Step 12: Check whether the units change requirements.md, stories.md or the application design files, and report it
- [x] Step 13: Present the completion message for team approval. Do not start Construction
