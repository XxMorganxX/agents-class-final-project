# Application Design Plan

**Stage:** INCEPTION - Application Design
**Project:** Anti-Vibe Coding (Pi, the learning sidecar)
**Depth:** Standard (execution-plan.md)
**Inputs read:** `inception/requirements/requirements.md` (approved 2026-11-23), `inception/user-stories/stories.md` and `personas.md` (approved 2026-11-24), `inception/plans/execution-plan.md` (approved 2026-11-24), `docs/technical-environment.md`, `docs/founding-idea.md`
**Outputs:** `inception/application-design/components.md`, `component-methods.md`, `services.md`, `component-dependency.md`, `application-design.md`

## How to use this file

1. **Answer the questions** in the Questions section. Write the letter of your choice after each `[Answer]:` tag. If no option fits, pick the last option (Other) and describe what you want. You can add a short note after a letter, for example `[Answer]: B, but keep the hook script under 20 lines`.
2. Answer every question. This stage asks more technical questions than earlier ones (how the code is split, how parts talk to each other, how it is tested), so the team's more technical members should lead these.
3. Tell the agent when you're done. The agent checks the answers for gaps and contradictions, and may write a short clarification file.
4. **Approve the plan,** or ask for changes. Then the agent writes the design artifacts.

## What this plan leaves alone

These are already decided, and no question reopens them:

- **Stack** (tech-env §2): TypeScript strict, Node 24, Electron with Forge and Vite, Fastify 5 in the main process, TypeBox, React 19, the pi agent harness `@earendil-works/pi-coding-agent` 1.4.0 with Haiku 5.5, Vitest, Testing Library, Playwright, fast-check. The prohibited libraries in tech-env §6 stay prohibited.
- **Behavior** (requirements FR-0 to FR-8): the agent's tools, the nine lesson component types, the teaching flow, grading and drill-down, the before-and-after check, the caps, and optional video.
- **Contracts that must exist** (NFR-7.1): `TutorModel`, Builder bridge, video runner, component registry. One config module (NFR-7.2).
- **Execution-plan points already settled by requirements:** hooks go in the user-level `~/.claude/settings.json`, never in the student's repo (FR-1.4 overrides tech-env §2). The tech-env §9 questions about blocking and window placement are answered (FR-1.6, FR-3.4).

The questions are about **how the parts are cut and wired**: component boundaries, interfaces, orchestration, communication and test seams. Detailed business rules (label rules, grading rules, cadence rules) come later in Functional Design, per unit.

## Candidate components (starting point for the questions)

This is the draft list the agent will refine from your answers. Names are working names.

| # | Candidate | Runs in | Main job |
|---|---|---|---|
| C1 | Config | shared | Every prompt, cap, timeout and the concept list, validated at start (US-33) |
| C2 | Schemas | shared | TypeBox schemas for hook payloads, IPC messages, model output and components |
| C3 | Hook Server | main | Fastify on `127.0.0.1`, token check, payload validation, `cwd` filter (US-06) |
| C4 | Hook Installer | main | Preview, merge, remove Pi's entries in `~/.claude/settings.json` (US-04, US-05) |
| C5 | Setup Checks | main | API key, Claude Code login, repo exists (US-01, US-02, US-03) |
| C6 | Builder Events | main | Turn raw hook payloads into typed events (file changed, turn ended, prompt sent) |
| C7 | Tutor Agent | main | pi harness session, tools, run limits, decides what to teach (US-11, US-12) |
| C8 | Repo Read Tools | main | `read`, `grep`, `find`, `ls` with path and secret filtering (US-09) |
| C9 | TutorModel | main | Contract over pi's model runtime; fake for tests |
| C10 | Builder Bridge | main | Read-only `claude -p` answer-key checks (US-24) |
| C11 | Component Registry | shared + renderer | Type definitions: schema, response schema, renderer, grader (US-13) |
| C12 | Graders | main | Right/wrong and checklist grading, drill-down steps (US-18 to US-20, US-25) |
| C13 | Learning Record | main | Per-concept history, mastery labels, before-and-after, metrics (US-30, US-31) |
| C14 | Budget / Caps | main | Haiku dollars, Builder calls, videos (US-27) |
| C15 | Video Runner | main | Optional `make-video` queue and `pi-video://` protocol (US-V1 to V3) |
| C16 | Event Log | main | Metadata JSONL log, debug content gate (US-32) |
| C17 | Preload API | preload | Small typed `contextBridge` API (US-10) |
| C18 | Window UI | renderer | Setup screen, lesson feed, settings, status notices (US-13, US-16, US-17) |

---

# Questions

## Part A: Component identification and code organization

## Question 1
How should the source tree be organized?

*Why we ask:* tech-env §2 sketches a feature layout with an `exercises/` folder per interaction type and `renderer/` outside `src/`. FR-2 now makes every card a lesson component in one registry, so the `exercises/` folder may not fit (execution-plan point 2). The layout also decides how teammates split the work in Units Generation.

A) Feature folders by process, as tech-env §2 but with `exercises/` replaced by `components/` (one folder per component type holding its schema, grader and renderer): `src/main/...`, `src/preload/`, `src/renderer/`, `src/shared/`

B) Feature folders not split by process: `src/tutor/`, `src/hooks/`, `src/components/<type>/`, `src/learning-record/` and so on, each holding whatever main, shared and renderer code it needs, with lint rules stopping renderer code from importing main-only code

C) Layered: `src/domain/` (pure rules), `src/adapters/` (pi harness, `claude`, Fastify, file system), `src/app/` (orchestration), `src/ui/`

D) Keep the tech-env §2 layout exactly, including `exercises/`

E) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
Where should each lesson component type's parts live, given that the schema is needed in main and renderer, the grader only in main, and the renderer only in the window?

A) One folder per type with three files (`schema.ts` shared, `grader.ts` main, `Card.tsx` renderer). A main-side registry and a renderer-side registry each import their half, keyed by the same type name

B) Schemas for all types in one shared file, graders in one main folder, renderers in one renderer folder. Simpler imports, but adding a type touches three folders

C) Everything for a type in one module that both processes import, with the renderer and grader loaded lazily

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
Should the before-and-after check and the session summary be their own component, or part of the Learning Record?

*Why we ask:* FR-5 (predictions and actual results) and FR-6.5 (four metrics) both read the record, but FR-5 also decides when to show "after" cards at End session, which is orchestration, not storage.

A) Separate: a Learning Record that only stores and labels, plus a Session Review component that runs the End-session flow, pairs predictions with results and computes the metrics

B) All in the Learning Record: storage, labels, prediction pairing and metrics. The End-session flow lives in the orchestrator

C) Metrics and pairing are pure functions next to the record; the End-session flow is one more trigger handled by the Tutor Agent like any other

D) Other (please describe after [Answer]: tag below)

[Answer]: C, End session already triggers an agent run to run the afters, and pure functions are pretty easy to property test

## Part B: Component methods and interface contracts

## Question 4
How should the agent produce a lesson component?

*Why we ask:* FR-0.3 gives the agent a "show lesson component" tool. The tool's input could be the finished component (the agent writes every field) or a request that a typed generator turns into a component with its own model call. This choice drives cost per card, the AC-4 answer-key quality, and how testable each type is.

A) One `show_component` tool whose parameters are a union of all registered component schemas. The agent fills every field itself in that tool call. The tool validates, sends flagged questions to the Builder, then shows the card

B) One tool per component type (`show_multiple_choice`, `show_explanation`, ...), each with that type's schema as its parameters, generated from the registry so adding a type needs no agent change

C) The agent only picks the concept, type and target code (`plan_lesson`). A per-type generator then makes a separate, focused `TutorModel.generateJson` call to write the component, as in the tech-env §8 example

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
What should the `TutorModel` contract cover?

*Why we ask:* NFR-7.1 puts a `TutorModel` in front of the model. The pi harness runs its own agent loop, so the contract could wrap only one-shot JSON calls, or the whole agent session too.

A) Two methods: `generateJson(schema, request)` for one-shot structured calls (grading, generators), and `runAgent(trigger, tools)` wrapping the pi agent session. The fake implements both with scripted outputs

B) Only `generateJson`. The agent session is created through pi's SDK directly, and tests swap pi's model provider for a fake one

C) A single `runAgent` method. Grading and generation also run as small agent calls

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
How should a student response travel from a card back to grading?

A) The window sends `{ cardId, response }` through one preload method. Main validates it against that type's response schema, grades it, updates the record, then starts an agent run with the result as its trigger (FR-0.2)

B) Same as A, but deterministic follow-ups (right/wrong feedback, "Show me the answer", drill-down step counting) are handled without starting an agent run. The agent runs only when it has to write new content, such as a drill-down question or retest

C) The window grades right/wrong answers itself (it already has the answer key) and sends only the result to main; checklist grading goes to main

D) Other (please describe after [Answer]: tag below)

[Answer]: B, but an agent run still starts after grading when the next step needs new content, so its not doing work outside our specified 3 circumstances

## Part C: Service layer and orchestration

## Question 7
What coordinates triggers, agent runs, cadence and caps?

*Why we ask:* Triggers arrive from three places (hooks, responses, End session). FR-3.3 says exercises wait for `Stop` with at most two per turn, explanations may appear live, and caps stop everything. Something has to own those rules.

A) A Session Orchestrator service: receives every trigger, applies cadence and caps, queues agent runs, and is the only thing that writes to the feed and the record

B) The Tutor Agent owns it all: triggers go straight to the agent, which reads cadence and cap state through tools and decides itself

C) A small event bus: each component subscribes to the events it cares about (hook events, responses, cap reached), with no central coordinator

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
How should agent runs be scheduled when triggers arrive faster than runs finish (Claude often makes several edits in a few seconds)?

A) One run at a time in a queue. Triggers that pile up while a run is busy are merged into the next run (for example, five file edits become one "files changed" trigger)

B) One run at a time; a new hook trigger cancels a run that hasn't shown anything yet, and responses always run

C) Separate lanes: one for live explanations during the turn, one for end-of-turn exercises and responses, each one at a time

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
Where should the Builder answer-key check (FR-4.5) happen?

A) Inside the show-component pipeline: a flagged component is held, the Builder Bridge is called, and the card is shown or dropped. The agent run has already ended

B) As an agent tool call (`ask_builder`) the agent makes before showing the card, so the agent can fix the key from the Builder's answer

C) Both: the agent may call `ask_builder` while writing, and the pipeline still checks any card that remains flagged

D) Other (please describe after [Answer]: tag below)

[Answer]: C, and a card already confirmed with ask_builder isn't checked a second time, so it doesn't use two Builder calls

## Part D: Dependencies and communication

## Question 10
How should main send cards and status to the window?

A) Push events: `card:added`, `card:updated` (graded, result added, code changed), `status:changed`. The window keeps its own feed list

B) State snapshots: main owns the full feed state and pushes the whole view model on every change. The window just renders it

C) Push events as in A, plus a `getSnapshot()` call the window uses on load or reload to rebuild the feed

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 11
How should the hook command post events to Pi, given FR-1.7 (exits at once when Pi is closed) and US-07 (under 200 ms, status 0, no output)?

A) An inline `curl -s --max-time 1 ... || true` command in the hook entry, as in tech-env §8, with the token in the command itself

B) A small Node script shipped in Pi's app-data folder; the hook entry runs `node <script>`, which reads the token and port from a file Pi writes at startup and exits quietly if Pi isn't listening

C) A tiny shell script in Pi's app-data folder that wraps `curl` the same way, so the entries in `settings.json` never change between sessions while the token does

D) Other (please describe after [Answer]: tag below)

[Answer]: B, the script always exits 0 with no output and a hard timeout under 200 ms

## Question 12
How should components get their dependencies, so tests can use the fake `TutorModel`, recorded fixtures and a fake Builder?

A) Constructor injection with one composition root (`src/main/app.ts`) that builds every real component; tests build the same graph with fakes

B) A small service container keyed by contract name

C) Module-level singletons, with Vitest module mocks in tests

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Part E: Design patterns and data

## Question 13
How should the session learning record be stored and changed?

*Why we ask:* FR-6.2 needs fixed label rules, US-30 asks for property tests on them, and tech-env §6 suggests in-memory state plus a JSONL event log.

A) Event-sourced: every answer, grade, skip and dispute is an event appended to an in-memory list (and mirrored to the JSONL log). Labels, prediction pairs and metrics are pure functions over the event list

B) A mutable in-memory object per concept, updated by methods; the JSONL log is only for debugging

C) Event-sourced as in A, but with a cached per-concept view updated on each event so reads stay cheap

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 14
How should the code snapshot behind a question be kept (FR-3.6, US-23: grading uses the code as it was when asked)?

A) Each card stores the snippet and file hash it was built from; grading uses the stored snippet, and a file watcher or the next hook event marks the card "code changed"

B) Each card stores only file path and line range; Pi re-reads the file at grading time and compares a stored hash to detect the change

C) Other (please describe after [Answer]: tag below)

[Answer]: A, Bs bad cause it cant recover the old code once the file changes (FR-3.6)

## Question 15
How separate should the optional video feature be?

A) A plug-in: the Video Runner registers its tool, its component type and its settings only when enabled and its checks pass. With it off, no video code runs and the core never imports it

B) Always loaded, behind a feature flag checked at each call site

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

# Answer analysis (Step 3)

Checked 2026-11-25. All 15 questions are answered. One contradiction needs the team (see `application-design-clarification-questions.md`). The other points below are settled within your answers; say so at approval if any reading is wrong.

| # | Point | How the design reads your answers |
|---|---|---|
| 1 | **Q11 B vs NFR-5.3 and US-04** | **Contradiction.** The token moves from the hook command into a runtime file. Clarification Question 1 |
| 2 | Q1 A splits code by process, and Q2 A puts all three files of a component type in one folder | `src/components/<type>/` is the one folder that holds code for several processes, as Q1 A itself describes. Lint and tsconfig rules keep `grader.ts` out of the renderer bundle and `Card.tsx` out of main |
| 3 | Q3 C (End session is an agent trigger) and US-27 (End session and the summary still work once the Haiku cap is reached) | When the Haiku cap is reached, the orchestrator skips the agent run for End session and shows the summary at once. Concepts without an "after" card show "after: not asked (cap reached)". Exact wording is left to Functional Design |
| 4 | Q6 B: which steps run without the agent | Deterministic: right/wrong feedback, **Show me the answer**, skip, dispute, drill-down step counting, adding an actual result. Agent run (the student-response trigger, FR-0.2): the "partly" follow-up, drill-down questions and the retest. Checklist grading of a teach-back is a one-shot `generateJson` call by the grader, not an agent run, and counts toward the Haiku cap |
| 5 | Q9 C: how the pipeline knows a card was already confirmed | `ask_builder` returns a verification receipt (id plus a hash of the question and answer key). The card carries the receipt id. The pipeline skips its own check only when the receipt says "agrees" and its hash matches the card's final question and key. If the agent changed the key after asking, the pipeline checks again |
| 6 | Q14 A says "a file watcher or the next hook event" | No file watcher. The next `PostToolUse` event for that file marks open cards "code changed". At grading time the grader compares the stored file hash with a fresh read, which also catches edits the student made by hand. Grading always uses the stored snippet |
| 7 | Q8 A: what gets merged | Only Builder hook triggers that pile up while a run is busy are merged into one "Builder activity" trigger (changed files, prompts sent, and whether the turn ended). Student responses and End session are never merged and keep their order |
| 8 | Q4 A with Q15 A: the `show_component` union must include video types only when video is on | The union is built from the registry when an agent session starts, so the video plug-in's types appear only after it registers them (FR-2.6) |
| 9 | Q5 A and the Haiku cap | `TutorModel` is wrapped by a budget decorator that reserves before each call and charges actual tokens after, so no call can skip the cap. Builder calls and videos are charged by the Builder Bridge and the video plug-in through the same ledger |
| 10 | Q11 B: what if Pi's script is gone but the hook entries remain | The entry is written so a missing script still exits 0 with no output. Exact command text is left to Functional Design for the Hook Installer |

---

# Execution Plan

Steps that depend on an answer name the question.

- [x] Step 1: Analyze context (requirements, stories, execution plan, tech env)
- [x] Step 2: Write this plan with questions
- [x] Step 3: Collect answers and check them for gaps and contradictions; write a clarification file if needed
- [x] Step 4: Get plan approval
- [x] Step 5: Generate `components.md` with component definitions, responsibilities and interfaces (Q1, Q2, Q3, Q15)
- [x] Step 6: Generate `component-methods.md` with method signatures and input/output types; business rules left to Functional Design (Q4, Q5, Q6, Q14)
- [x] Step 7: Generate `services.md` with service definitions and orchestration patterns (Q7, Q8, Q9)
- [x] Step 8: Generate `component-dependency.md` with the dependency matrix, communication patterns and data-flow diagrams, Mermaid checked by hand with text alternatives (Q10, Q11, Q12, Q13)
- [x] Step 9: Generate `application-design.md` consolidating the four files
- [x] Step 10: Validate completeness and consistency: every story in stories.md is served by at least one component; every NFR-7.1 contract is defined; no component writes to the student's repo
- [x] Step 11: Update `aidlc-state.md` and `audit.md`, and present the completion message
