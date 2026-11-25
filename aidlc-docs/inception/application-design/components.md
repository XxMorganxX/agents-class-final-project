# Components

**Stage:** INCEPTION - Application Design
**Sources:** `application-design-plan.md` (answers Q1-Q15 and the answer analysis), `application-design-clarification-questions.md` (clarification 1), requirements.md, stories.md, docs/technical-environment.md

This file names each component, says what it is responsible for, and lists its interface at a high level. Method signatures are in `component-methods.md`. Orchestration is in `services.md`. Dependencies and data flow are in `component-dependency.md`.

## Design decisions that shape the components

| Decision | Source | Effect on components |
|---|---|---|
| Feature folders split by process: `src/main/`, `src/preload/`, `src/renderer/`, `src/shared/` | Q1 A | Each component sits in one process folder. Main-only code never reaches the window bundle |
| One folder per lesson component type, `src/components/<type>/`, holding `schema.ts` (shared), `grader.ts` (main) and `Card.tsx` (renderer) | Q1 A, Q2 A | The only folder that holds code for more than one process. Lint and tsconfig rules keep `grader.ts` out of the renderer and `Card.tsx` out of main |
| Prediction pairing, labels and metrics are pure functions next to the record. End session is one more agent trigger | Q3 C | The Learning Record has a small store plus a pure rules module. There is no separate Session Review component |
| The agent fills every field of a component in one `show_component` call, whose parameters are a union built from the registry | Q4 A | The Tutor Agent has no per-type generators. The registry builds the tool schema |
| `TutorModel` has `generateJson` and `runAgent` | Q5 A | One contract covers the pi agent session and one-shot calls such as checklist grading |
| Deterministic follow-ups run without the agent. An agent run starts only when the next step needs new content | Q6 B | The Session Orchestrator owns feedback, skip, dispute, "Show me the answer" and drill-down step counting |
| A Session Orchestrator receives every trigger and is the only writer to the feed and the record | Q7 A | One service in main coordinates everything (see `services.md`) |
| One agent run at a time. Builder hook triggers that pile up are merged | Q8 A | A run queue inside the orchestrator |
| `ask_builder` while writing, plus a pipeline check for cards still flagged. A card already confirmed by a matching receipt isn't checked twice | Q9 C | The Card Pipeline checks receipts by a hash of the question and answer key |
| Push events plus `getSnapshot()` | Q10 C | The IPC Gateway sends card and status events. The window can rebuild its feed on reload |
| Hook entries run a Node script. The script reads the token and port from a `0600` runtime file that Pi rewrites with a new token at each launch. It always exits 0 with no output, with a hard timeout under 200 ms | Q11 B, clarification 1 | The Hook Installer writes fixed entries once. The token never appears in `settings.json` |
| Constructor injection with one composition root | Q12 A | `src/main/app.ts` builds the real graph. Tests build the same graph with fakes |
| The learning record is event-sourced. Labels, pairs and metrics are pure functions over the events | Q13 A | Property tests run on pure functions with no I/O |
| Each card stores its snippet and file hash. No file watcher | Q14 A, analysis 6 | The Card Pipeline takes a `CodeSnapshot` for every card that refers to code |
| Video is a plug-in that registers its tool, its types and its settings only when it is on and its checks pass | Q15 A | The core never imports `src/plugins/video/` |

## Source layout

```
src/
  shared/
    config.ts               C1  every prompt, cap, timeout, the concept list (no secrets)
    schemas/                C2  hook payloads, IPC messages, model output, common types
  components/
    registry/               C3  registry contract and the main and renderer halves
    explanation/            C4  schema.ts, grader.ts (none for display-only), Card.tsx
    multiple-choice/
    written-response/
    quick-time/
    system-design-graph/
    data-graphic/
    prediction/
    session-summary/
  main/
    app.ts                  C22 composition root
    env.ts                      reads .env (part of C7)
    hook-server/            C5
    hook-installer/         C6  plus the hook script asset and runtime file writer
    setup/                  C7
    builder-events/         C8
    session/                C9, C10  orchestrator, run queue, cadence, recovery, card pipeline
    tutor/                  C11, C12  agent session, tools, TutorModel adapters
    repo-tools/             C13
    builder-bridge/         C14
    learning-record/        C15  event store and pure rules
    budget/                 C16
    event-log/              C17
    window/, ipc/           C18
  preload/                  C19
  renderer/                 C20
  plugins/
    video/                  C21 optional; main runner, video types, pi-video:// protocol
resources/
  pi-hook.mjs                   hook script copied into Pi's app-data folder (C6)
test/
  fakes/                        FakeTutorModel, FakeBuilderBridge, FakeVideoRunner, fake clock
  fixtures/hooks/               recorded Claude Code hook payloads
```

## Component catalog

| ID | Component | Process | Kind |
|---|---|---|---|
| C1 | Config | shared | Module |
| C2 | Shared Schemas | shared | Module |
| C3 | Component Registry | shared, main, renderer | **Contract** (NFR-7.1) |
| C4 | Lesson Component Types | shared, main, renderer | Module per type |
| C5 | Hook Server | main | Adapter (HTTP in) |
| C6 | Hook Installer | main | Adapter (file system) |
| C7 | Setup Checks | main | Adapter |
| C8 | Builder Events | main | Pure module |
| C9 | Session Orchestrator | main | **Service** |
| C10 | Card Pipeline | main | **Service** |
| C11 | Tutor Agent | main | Service |
| C12 | TutorModel | main | **Contract** (NFR-7.1) |
| C13 | Repo Read Tools | main | Adapter (file system, read-only) |
| C14 | Builder Bridge | main | **Contract** (NFR-7.1) |
| C15 | Learning Record | main | Store and pure rules |
| C16 | Budget Ledger | main | Store and pure rules |
| C17 | Event Log | main | Adapter (file system) |
| C18 | Window Shell and IPC Gateway | main | Adapter (Electron) |
| C19 | Preload API | preload | Contract |
| C20 | Renderer App | renderer | UI |
| C21 | Video Plug-in | main, renderer | Optional plug-in; holds the **Video Runner contract** (NFR-7.1) |
| C22 | Composition Root | main | Wiring |

---

## C1 Config
- **Purpose:** the single source of truth for every prompt and tunable value (NFR-7.2, NFR-1.4).
- **Responsibilities:**
  - Caps: Haiku $0.25, 15 Builder calls, 3 videos. Timeouts, agent step limit, exercises per turn (2), quick-time countdown, latency targets (15 s and 60 s).
  - The fixed concept list (about 20 concepts) with ids and short definitions.
  - Every prompt: the agent system prompt, trigger messages, checklist grading prompt, Builder question template.
  - `VIDEO_ENABLED_BY_DEFAULT = false`.
  - Validates itself at start. An invalid value stops Pi and names the value (US-33).
- **Interface:** `CONFIG` (read-only object) and `validateConfig()`.
- **Never holds:** secrets. `ANTHROPIC_API_KEY` and `VIDEO_ENGINE_DIR` come from `.env` through C7.
- **Stories:** US-33, US-27, US-16, US-20.

## C2 Shared Schemas
- **Purpose:** TypeBox schemas for every boundary (NFR-5.6).
- **Responsibilities:** hook payloads (`PostToolUse`, `Stop`, `UserPromptSubmit`), the runtime file, IPC messages in both directions, the agent's tool parameters other than `show_component`, and common types (`CardId`, `ConceptId`, `CodeSnapshot`, `Grade`).
- **Interface:** exported schemas, their `Static` types, and one `check(schema, value)` helper that returns a result instead of throwing.
- **Stories:** US-06, US-13, US-10.

## C3 Component Registry (contract)
- **Purpose:** the one place that knows which lesson component types exist (FR-2.2, FR-2.6).
- **Responsibilities:**
  - Register a type: its name, field schema, response schema (or none), whether it is an exercise, whether it refers to code, and its grader (main) or card renderer (renderer).
  - Validate a component's fields and a student's response against the type's schemas.
  - Build the `show_component` parameter schema as a union of every registered type (Q4 A). The union is built when an agent session starts, so plug-in types appear only once registered (analysis 8).
  - The main half holds graders. The renderer half holds card renderers. Both are keyed by the same type name.
- **Interface:** `ComponentRegistry` (main) and `CardRegistry` (renderer). See `component-methods.md`.
- **Stories:** US-13, US-12, US-10.

## C4 Lesson Component Types
- **Purpose:** the eight core types from FR-2.7. Video types live in C21.
- **Responsibilities, per type folder:**
  - `schema.ts`: the fields the agent fills in and the response schema.
  - `grader.ts` (types the student answers): right/wrong for multiple choice and quick-time; checklist grading for written response through `TutorModel.generateJson`; prediction pairing with the actual result.
  - `Card.tsx`: renders fields as text only (no HTML from the model), plus the card's own controls (answer, Skip, Dispute, Show me the answer, Drill down).
- **Types:** explanation, multiple-choice, written-response, quick-time, system-design-graph (Mermaid, falls back to a code block), data-graphic (bar, line, table from inline data), prediction (before or after, with an actual-result field), session-summary.
- **Stories:** US-14, US-15, US-18, US-19, US-20, US-21, US-29, US-31.

## C5 Hook Server
- **Purpose:** receive Claude Code hook events (FR-1.5).
- **Responsibilities:**
  - Fastify on `127.0.0.1` only. Port from config.
  - Reject events without this session's token (`x-pi-token`), and payloads that fail their schema. Log each rejection.
  - Reply `202` at once, before any teaching work, and pass the payload to C8 and then the orchestrator.
- **Interface:** `HookServer.start()`, `HookServer.stop()`.
- **Stories:** US-06, US-07.

## C6 Hook Installer
- **Purpose:** add and remove Pi's entries in the user-level `~/.claude/settings.json` (FR-1.4, FR-1.7).
- **Responsibilities:**
  - Show the exact entries before changing anything. Write nothing until the student confirms.
  - Merge Pi's entries into the existing settings and leave everything else as it was. Refuse to write if the file isn't valid JSON.
  - Keep a byte-exact backup of the file taken just before install. On removal, restore the backup when nothing else changed; otherwise remove only Pi's entries (US-05, AC-6). Exact rules in Functional Design.
  - Copy `pi-hook.mjs` into Pi's app-data folder. The entries run `node <that script> <EventName>`, and never contain the token (clarification 1). If the script is missing, the entry still exits 0 with no output (analysis 10).
  - At each launch, write the runtime file (`port`, new random `token`, `pid`) with mode `0600`, and delete it when Pi quits (clarification 1).
  - The merge and remove logic is pure (JSON in, JSON out) so it can be property-tested.
- **Interface:** `preview()`, `install()`, `remove()`, `status()`, `writeRuntimeFile()`, `deleteRuntimeFile()`.
- **Hook script (`pi-hook.mjs`):** reads stdin and the runtime file, posts to `127.0.0.1` with the token, hard timeout under 200 ms, always exits 0 with no output (Q11).
- **Never touches:** any file in the student's repo.
- **Stories:** US-04, US-05, US-07.

## C7 Setup Checks
- **Purpose:** the four first-run checks (FR-8) and reading `.env` (FR-8.1).
- **Responsibilities:**
  - Read `ANTHROPIC_API_KEY` in main only. Report a missing or empty key by variable name, never by value.
  - Store and check the chosen repo path in Pi's app-data folder (FR-1.3).
  - Check Claude Code is installed and logged in (`claude auth status`, argument array).
  - Check the API key works with one tiny call.
  - Hand the hooks step to C6.
- **Interface:** `runStep(step)`, `runAll()`, `chooseRepo(path)`, `currentRepo()`.
- **Stories:** US-01, US-02, US-03.

## C8 Builder Events
- **Purpose:** turn raw hook payloads into typed Builder events.
- **Responsibilities:**
  - Map `PostToolUse` (Edit, Write) to `FileChanged`, `Stop` to `TurnEnded` (with the end-of-turn message and test commands from the transcript), `UserPromptSubmit` to `PromptSent`.
  - Drop events whose `cwd` is outside the chosen repo and log only that they were ignored (FR-1.3).
  - Merge a list of pending events into one `BuilderActivity` trigger (Q8 A, analysis 7).
- **Interface:** `toBuilderEvent(payload, repoRoot)`, `mergeActivity(events)`. Both pure.
- **Stories:** US-06, US-03, US-21 (test results from the transcript).

## C9 Session Orchestrator (service)
- **Purpose:** coordinate one Pi session (Q7 A). Detailed in `services.md`.
- **Responsibilities:**
  - Receive every trigger: Builder activity, student response, End session.
  - Queue agent runs one at a time and merge Builder triggers that pile up (Q8 A).
  - Apply cadence: explanations may show live; exercises only after `Stop`, at most two per turn (FR-3.3). Rules are pure functions in `session/cadence.ts`.
  - Handle deterministic follow-ups without the agent: feedback, skip, dispute, "Show me the answer", drill-down step counting, adding an actual result (Q6 B, analysis 4). Recovery steps are pure functions in `session/recovery.ts`.
  - Check caps before each run. Show the cap notice once. Run End session without Haiku when the cap is reached (analysis 3).
  - Be the only writer to the feed and the learning record.
  - Show and clear the "preparing" indicator and the model-failure notice (NFR-2.3, NFR-3.1).
- **Interface:** `handleBuilderEvent()`, `handleResponse()`, `handleCardAction()`, `endSession()`, `snapshot()`.
- **Stories:** US-08, US-11, US-12, US-16, US-17, US-22, US-25, US-26, US-27, US-28, US-29.

## C10 Card Pipeline (service)
- **Purpose:** turn a component the agent proposed into a card on screen, or drop it (FR-2.4, FR-4.5).
- **Responsibilities:**
  - Validate against the registry. On failure, ask the agent to regenerate once, then drop and log (FR-2.4).
  - Take a `CodeSnapshot` (snippet, line range, file hash) for every component that refers to code (Q14 A).
  - Answer-key check: skip when the card carries a receipt that says "agrees" and whose hash matches the card's final question and key; otherwise, if flagged, call the Builder Bridge and show or drop the card (Q9 C, analysis 5).
  - Enforce the latency limit: drop a card that isn't ready within 15 s (60 s when Builder-checked) and log it (US-17).
  - Hand the finished card to the orchestrator.
  - Mark open cards "code changed" when a later `FileChanged` event names their file (analysis 6).
- **Interface:** `propose()`, `markCodeChanged()`.
- **Stories:** US-13, US-17, US-23, US-24.

## C11 Tutor Agent
- **Purpose:** the teaching assistant agent on the pi harness (FR-0).
- **Responsibilities:**
  - Build the agent request for each trigger: system prompt from config, a trigger message, the session's learning record summary, and the tools.
  - Register the tools: C13's four read tools, `ask_builder` (C14), `show_component` (C3 union, handled by C10), and `offer_video` only when C21 is active (FR-0.3).
  - End a run on the first shown component, "nothing to teach", the step limit or a cap (FR-0.4). Nothing half-finished is shown (US-11).
  - Tag concepts only from the config list. Drop tags that aren't on it (FR-3.1).
  - Report every run to C17 (FR-0.6).
- **Interface:** `run(trigger, context)`.
- **Stories:** US-11, US-12, US-09.

## C12 TutorModel (contract)
- **Purpose:** the only way Pi reaches a model (NFR-7.1, Q5 A).
- **Responsibilities:**
  - `generateJson`: one-shot structured call checked against a schema (checklist grading, prediction judging, the API key check).
  - `runAgent`: one pi agent session run with the given tools, step limit and timeout.
  - Brief retries on failure, then a typed failure the orchestrator turns into a notice (NFR-3.1).
- **Implementations:**
  - `PiTutorModel`: pi `ModelRuntime` with the `anthropic` provider and `claude-haiku-5-5`. Never imports `@anthropic-ai/sdk` directly (tech-env §6).
  - `BudgetedTutorModel`: a decorator that reserves budget before a call and charges actual tokens after, so no call skips the Haiku cap (analysis 9).
  - `FakeTutorModel` (tests): scripted outputs and tool calls.
- **Stories:** US-11, US-27, US-28; every story through the definition of done.

## C13 Repo Read Tools
- **Purpose:** the agent's read-only code tools (NFR-5.4).
- **Responsibilities:**
  - `read`, `grep`, `find`, `ls`, rooted at the chosen repo.
  - A path guard refuses paths outside the repo (after resolving `..` and symlinks) and secret files (`.env*`, `*.pem`, `id_*`, `.git/`), and logs each refusal. Registered as pi's `tool_call` handler.
  - The path guard is a pure function, so it can be property-tested.
  - Also used by C10 to read snippets and file hashes.
- **Interface:** `createRepoTools(repoRoot)`, `guardPath(repoRoot, requested)`.
- **Stories:** US-09.

## C14 Builder Bridge (contract)
- **Purpose:** read-only questions to Claude Code (FR-4.5, NFR-5.5).
- **Responsibilities:**
  - Run `claude -p` in the repo with `--allowedTools Read,Grep,Glob` and `--permission-mode dontAsk`, as an argument array (NFR-5.7), with a timeout.
  - `checkAnswerKey`: ask whether a question's answer key is right for the code; return a verdict and a receipt holding a hash of the question and key (analysis 5).
  - Reserve a Builder call in C16 before each call. At the cap, return "cap reached" without calling.
  - Keep receipts for the session so C10 can look them up.
  - A failure returns a typed result and never throws into the teaching loop (NFR-3.2).
- **Implementations:** `ClaudeCliBuilderBridge`, `FakeBuilderBridge` (tests).
- **Stories:** US-24, US-09, US-27.

## C15 Learning Record
- **Purpose:** the session's learning state and the agent's only working state (FR-0.5, FR-6, Q13 A).
- **Responsibilities:**
  - Store: an append-only in-memory list of learning events (question asked, answered, graded, skipped, disputed, prediction made, actual result added, explained, card closed), mirrored to the event log. Empty at every launch (FR-6.4).
  - Pure rules (`learning-record/rules.ts`): mastery label per concept (not yet shown, shaky, solid; "solid" needs two exercise types), the next lesson hint per concept (diagnose first or explain first, FR-3.2), prediction pairs, the four metrics, and the session summary fields.
- **Interface:** `LearningRecord.append()`, `events()`, plus the pure functions in `component-methods.md`.
- **Stories:** US-08, US-12, US-21, US-22, US-26, US-29, US-30, US-31.

## C16 Budget Ledger
- **Purpose:** session caps (NFR-1).
- **Responsibilities:**
  - Track Haiku dollars, Builder calls and videos against the config caps.
  - Reserve before spending and settle after, so the ledger never goes over a cap and never counts one call twice. The accounting is pure so it can be property-tested.
  - Announce once when a cap is reached.
- **Interface:** `tryReserve()`, `settle()`, `release()`, `snapshot()`, `onCapReached()`.
- **Stories:** US-27, US-24, US-V3.

## C17 Event Log
- **Purpose:** metadata logs, and full content only in debug mode (NFR-4).
- **Responsibilities:**
  - Append JSONL to Pi's app-data folder: run records (trigger, tool calls, outcome, timings, tokens), rejections, drops, cap events.
  - Write prompts, model outputs and student answers only when the debug setting is on.
  - A write failure never stops teaching; it is reported once (US-32).
- **Interface:** `logRun()`, `logEvent()`, `logContent()`, `onWriteFailure()`.
- **Stories:** US-32, US-11.

## C18 Window Shell and IPC Gateway
- **Purpose:** the Electron window and the main side of IPC.
- **Responsibilities:**
  - Create the window with `contextIsolation: true`, `sandbox: true`, `nodeIntegration: false`, a strict CSP, and blocked navigation and new windows (NFR-5.1). A normal movable window; no focus stealing (FR-3.4).
  - Register IPC handlers, check every incoming message against its schema, and forward it to the orchestrator, setup or installer.
  - Push `card:added`, `card:updated`, `status:changed` and `setup:changed` events (Q10 C).
- **Interface:** `createMainWindow()`, `IpcGateway.register()`, `IpcGateway.push()`.
- **Stories:** US-10, US-16.

## C19 Preload API
- **Purpose:** the only bridge from the window to main (NFR-5.2).
- **Responsibilities:** expose `window.pi` through `contextBridge` with the typed methods and event subscriptions in `component-methods.md`, and nothing else. Never exposes `ipcRenderer` or keys.
- **Stories:** US-10, US-01.

## C20 Renderer App
- **Purpose:** the React window.
- **Responsibilities:**
  - Setup screen with four steps and pass or fail per step (FR-8.2). Reopenable from settings.
  - Lesson feed: cards in order, rendered through the renderer half of C3. Rebuild from `getSnapshot()` on load (Q10 C).
  - Badge or highlight for a new card. No OS notifications (FR-3.4).
  - Status line: preparing indicator, model-failure notice, cap notice, videos left.
  - Settings: debug logging, videos toggle (shown only to start C21's checks), Remove Pi hooks, reopen setup.
  - **End session** button.
  - Renders model text as plain text only (US-10).
- **Stories:** US-02, US-13, US-16, US-17, US-29, US-31, US-V1.

## C21 Video Plug-in (optional)
- **Purpose:** optional video lessons through Project Feynman (FR-7). Holds the **Video Runner contract**.
- **Responsibilities:**
  - Check requirements when the student turns videos on: macOS with Metal, `uv`, Python 3.14, the checkout at `VIDEO_ENGINE_DIR`, logged-in Claude Code. Name what is missing (FR-7.2).
  - When on and checks pass, register `offer_video`, the `video-offer`, `video-rendering` and `video` component types, and the `pi-video://` protocol, which serves only files from the video output folder.
  - Run `make-video` with `--agent claude` as an argument array, without `ANTHROPIC_API_KEY` in its environment. One job at a time (FR-7.4).
  - Reserve a video in C16. Replace a failed render with a text explanation of the same concept (FR-7.6).
  - With videos off, none of its main-process code runs, and the main-process core doesn't import it; only the composition root loads it, and only after the checks pass (Q15 A). The renderer always bundles the video card renderers, which stay idle because main never sends a video card.
- **Interface:** `VideoPlugin.activate(context)`, `deactivate()`; `VideoRunner` contract.
- **Stories:** US-V1, US-V2, US-V3.

## C22 Composition Root
- **Purpose:** build the object graph once (Q12 A).
- **Responsibilities:** validate config, read `.env`, create every real component with its dependencies, start the hook server, write the runtime file, open the window, and shut down in reverse order. Tests call the same builder with fakes.
- **Interface:** `createApp(overrides?)`, `App.start()`, `App.stop()`.
- **Stories:** US-01, US-08; enables the test seams in every story.

---

## Story coverage

Every story in stories.md is served by at least one component.

| Story | Components |
|---|---|
| US-01 | C7, C19, C22 |
| US-02 | C7, C20 |
| US-03 | C7, C8 |
| US-04 | C6 |
| US-05 | C6, C20 |
| US-06 | C5, C8, C2 |
| US-07 | C5, C6 (hook script), C9 |
| US-08 | C15, C9, C22 |
| US-09 | C13, C14, C11 |
| US-10 | C18, C19, C20, C4 |
| US-11 | C11, C12, C17 |
| US-12 | C11, C15 (next lesson hint) |
| US-13 | C3, C4, C10, C20 |
| US-14 | C4 (explanation), C9 (cadence), C10 (snapshot) |
| US-15 | C4 (system-design-graph, data-graphic) |
| US-16 | C9 (cadence), C20 |
| US-17 | C9, C10, C20 |
| US-18 | C4 (multiple-choice), C9 |
| US-19 | C4 (written-response grader), C12 |
| US-20 | C4 (quick-time), C9 |
| US-21 | C4 (prediction), C15, C8 |
| US-22 | C9, C15 |
| US-23 | C10, C4 |
| US-24 | C14, C10, C16 |
| US-25 | C9 (recovery), C11 |
| US-26 | C9, C15 |
| US-27 | C16, C12 (budget decorator), C9 |
| US-28 | C12, C9, C20 |
| US-29 | C9, C11, C4 (prediction) |
| US-30 | C15 |
| US-31 | C15, C4 (session-summary) |
| US-32 | C17 |
| US-33 | C1 |
| US-V1 | C21, C20 |
| US-V2 | C21 |
| US-V3 | C21, C16 |
