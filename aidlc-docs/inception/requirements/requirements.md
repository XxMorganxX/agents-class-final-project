# Requirements: Anti-Vibe Coding (Pi, the learning sidecar)

**Stage:** INCEPTION - Requirements Analysis
**Depth:** Comprehensive
**Sources:** `docs/founding-idea.md`, `docs/technical-environment.md`, `README.md`, `docs/assignment-requirements.md`, and the team's answers in `requirement-verification-questions.md` (all 35 answered, 2026-11-20)

---

## 1. Intent analysis

| Item | Value |
|---|---|
| User request | Build Pi, a teaching assistant agent that runs as an Electron sidecar beside Claude Code (the Builder). It explains the code Claude writes and checks that the student understands it, without writing to the student's repo |
| Request type | New project (greenfield) |
| Scope estimate | System-wide: Electron app, local hook server, tutor agent, lesson component system, Builder bridge, optional video runner, session learning record |
| Complexity estimate | Complex |
| Build window | Two weeks of Construction (team direction) |

## 2. Summary of key requirements

- **Pi is a teaching assistant agent.** It runs on the pi agent harness with Haiku 5.5. On each trigger it decides what is worth teaching, reads the code to ground that choice, asks the Builder when it needs a check, and shows one lesson component. Its tools can only read.
- **Pi only explains and checks understanding.** It never changes the student's repo. It reads Claude Code's hook events and code, and shows lessons in its own window.
- **Everything Pi shows is a lesson component.** The window is a chat-style feed of typed, schema-checked components. The agent picks a component type from a fixed registry and fills in its fields. It cannot produce free-form UI.
- **Core teaching loop.** Live explanations, multiple choice, written teach-back, quick-time events, display-only diagrams and data graphics, then drill-down on wrong answers.
- **Before-and-after check is core.** A prediction component records the student's prediction before Pi teaches a concept, and another at session end, each with the actual result. The summary compares them and reports prediction, comprehension, interruption cost and cost-efficiency.
- **Answer keys are checked by the Builder.** Haiku flags questions whose answers depend on how the code runs. Pi holds those until a read-only `claude -p` call confirms them.
- **Videos are real but optional.** They are made by Project Feynman (make-video) through a background Claude session, and only when the runner and its requirements are on the machine. Pi works fully without them.
- **Cheap and local.** Haiku 5.5 for Pi's own work, a $0.25 Haiku cap per session, 15 Builder calls and 3 videos per session. Everything runs on the student's laptop.

## 3. Changes from the founding idea

The team changed these points during Requirements Analysis. The founding idea document itself is unchanged.

| Founding idea | Now | Decided by |
|---|---|---|
| §10 lists before-and-after checks under v2 | Core MVP feature | Q33, team direction |
| §10 lists pseudocode and system design exercises as in scope | No pseudocode or design answer input in the MVP. Diagrams are display-only. A written-response component may ask for pseudocode as plain text | Q3, Q9 |
| §10 "Pi → Claude: requests for explanations, answer verification, and concept identification" | MVP Pi → Builder calls are for answer verification only. Builder explanations and concept identification are stretch goals | Q3, Q25 |
| §6 calls the window a "chat window" | A chat-style feed of lesson components. Pi starts every exchange, and the student cannot type free questions | Q6, team direction 2 |
| §4 five mastery levels | Three levels: not yet shown, shaky, solid | Q19 |
| tech-env §7 keeps data in `.pi-sidecar/` (location unstated) | Pi's own app-data folder, never the student's repo | Q13, Q28 |
| tech-env §5 recorded hook fixtures | Kept for automated tests. No replay mode for the demo | Q35 |
| NFR-5.3 (this document): token put in the hook command | Token in a `0600` runtime file that Pi rewrites at each launch; the hook script reads it. Hook entries stay the same between sessions (helps FR-1.7 and AC-6) | Application Design Q11 and clarification 1, 2026-11-25 |

The README's scope section was updated on 2026-11-23 to match these changes. `founding-idea.md` is unchanged.

## 4. Users

- **Primary user: a CS student** at the intro-to-intermediate level, building a project with Claude Code. Pi pitches questions at the intermediate level and defines every technical term the first time it uses it. (Q1)
- **Student's code language:** JavaScript/TypeScript only in the MVP. (Q2)
- **Secondary user: a team member running the demo**, who installs Pi from the public repo, possibly without the video runner.

---

## 5. Functional requirements

### FR-0 Pi, the teaching assistant agent (team direction, 2026-11-20)

- **FR-0.1** Pi is a teaching assistant agent built on the pi agent harness (`@earendil-works/pi-coding-agent`), with Claude Haiku 5.5 as its model. The agent decides what to teach, when, and with which lesson component. Fixed rules handle grading labels, caps and the session record.
- **FR-0.2** Things that start an agent run:
  - a hook event: a file edit, the end of Claude's turn, or a new prompt from the student
  - the student's response to a component
  - the End session button
- **FR-0.3** The agent has these tools and no others:
  - **Code tools:** read-only `read`, `grep`, `find` and `ls`, limited to the chosen repo (NFR-5.4).
  - **`ask_builder`:** a read-only `claude -p` question to the Builder. In the MVP it is used to check answer keys (FR-4.5).
  - **Show lesson component:** shows a component of a registered type (FR-2).
  - **Offer video:** only available when videos are on and their requirements are met (FR-7).

  No tool writes files, runs code or sends anything to Claude Code's live session.
- **FR-0.4** An agent run ends when one of these happens:
  - it shows a component
  - it decides nothing is worth teaching right now
  - it reaches its step limit for one run
  - it reaches a session cap (NFR-1)

  The step limit and the time limit per call live in the config module (NFR-7.2).
- **FR-0.5** The session learning record (FR-6) is the agent's working state. The agent keeps no memory across sessions (FR-6.4).
- **FR-0.6** Every agent run is logged: what started it, the tool calls, the component it showed or its reason for showing nothing, plus timings and tokens (NFR-4).

### FR-1 Session and pairing with Claude Code

- **FR-1.1** A session is one launch of the Pi app. Learning state lasts until Pi closes, through Claude Code restarts and `/clear`. (Q14)
- **FR-1.2** The Pi window has an **End session** button. It runs the "after" half of the before-and-after check (FR-5), then shows the session summary (FR-6.3). (Q14, Q33)
- **FR-1.3** The student picks one repo in the Pi window. Pi ignores hook events whose `cwd` is outside that repo. (Q13)
- **FR-1.4** Pi's hooks are installed in the student's **user-level** Claude Code settings. No file in the student's repo is created or changed, now or later. (Q13)
- **FR-1.7** Installing and removing hooks (gap 2):
  - **Asks first.** Before installing, Pi shows the exact hook entries it will add and asks the student to confirm.
  - **Merges.** Pi adds its own entries and leaves every existing setting and hook exactly as it was.
  - **Removable.** A **Remove Pi hooks** button deletes only Pi's entries.
  - **Harmless when Pi is closed.** If Pi isn't running, each hook exits at once without an error, so Claude Code is never slowed or blocked.
- **FR-1.5** Pi receives `PostToolUse`, `Stop` and `UserPromptSubmit` hook events on its local server. It gets Claude's decisions and test results only from what Claude Code already produces: the end-of-turn message, the transcript and the test commands Claude ran. (Q11)
- **FR-1.6** Pi never blocks or delays Claude Code. Every exercise is about code Claude has already written. A "choose the architecture" exercise asks which approach the student would have picked, then compares that with what Claude built. (Q10)

### FR-2 Lesson component system (team direction 2)

- **FR-2.1** Everything Pi shows the student, and everything it asks of them, is a **lesson component** of a registered type.
- **FR-2.2** Each component type defines:
  - the fields the agent fills in, as a TypeBox schema
  - the student's response, as a schema, or none for display-only types
  - a renderer
  - an optional grader
- **FR-2.3** Pi's agent can only emit registered component types. It cannot emit HTML, scripts, styles or other free-form UI.
- **FR-2.4** Every component is checked against its schema before it is shown. An invalid component is regenerated once, then dropped and logged.
- **FR-2.5** The Pi window shows components in order as a chat-style feed. The student responds only through each component's own controls. (Q6)
- **FR-2.6** Adding a component type needs no change to the agent loop, the feed, grading flow or the session record.
- **FR-2.7** MVP component types:

| Type | Student responds? | Notes |
|---|---|---|
| Explanation | No | Text, plus a code snippet with file and line references |
| Multiple choice | Yes, picks one option | Right or wrong grading |
| Written response (teach-back) | Yes, free text | Checklist grading (FR-4.2) |
| Quick-time event | Yes, picks one option before a countdown ends | Multiple choice with a time limit. Timing out counts as skipped |
| System design graph | No | Mermaid text drawn by the window. Display only |
| Data graphic | No | A small chart format (bar, line or table) with data written inline. It cannot load anything from the network or run code |
| Video | No | Plays a generated MP4 (FR-7). Shown only when videos are enabled |
| Prediction | Yes, picks an option or writes a short prediction; later the actual result is added | Holds both halves of the before-and-after check (FR-5): the prediction, then the actual result, shown side by side |
| Session summary | No | Shown at session end (FR-6.3) |

### FR-3 Teaching flow

- **FR-3.1** Pi picks concepts from a fixed list of about 20, written by the team for the demo app. Haiku tags each change with concepts from that list only. (Q7)
- **FR-3.2** For a concept new to this session, Pi asks a diagnostic question first. For a concept the student already missed this session, Pi explains first. (Q4)
- **FR-3.3** Cadence (Q15):
  - Short explanations appear live while Claude edits significant files.
  - Exercises wait until the end of Claude's turn, with at most two per turn.
- **FR-3.4** Pi signals a new card only with a badge or highlight inside its own window. It sends no OS notifications and doesn't steal focus. The window is a normal window the student places anywhere. (Q16, Q17)
- **FR-3.5** The student can skip any exercise. A skip is recorded as "skipped", not wrong, and that question isn't asked again this session. (Q18)
- **FR-3.6** If the code a question refers to changes while the question is on screen, the question stays as it was asked. Grading uses the code from that moment, and the card notes that the code has since changed. (Q22)

### FR-4 Grading, drill-down and answer-key checks

- **FR-4.1** Multiple choice and quick-time events are graded right or wrong.
- **FR-4.2** Written responses are graded against a checklist. Pi drafts the checklist from the actual code in the same model call that writes the question. The verdict is "got it", "partly" or "not yet", and the student sees which points they hit and missed. "Partly" gets one targeted follow-up, and "not yet" starts a drill-down. (Q20)
- **FR-4.3** After a wrong answer, the student chooses one of two (Q5):
  - **Show me the answer.**
  - **Drill down:** up to two focused follow-ups and one retest with a new example, then Pi reveals the answer.
- **FR-4.4** The student can mark any grade "disputed" in the session record. There is no recheck. (Q21)
- **FR-4.5** Haiku flags questions whose correct answer depends on how the code runs. Pi holds each flagged question until the Builder confirms its answer key through read-only `claude -p`. The question is dropped if the Builder disagrees, times out, or the Builder-call cap is reached. (Q12)

### FR-5 Before-and-after comprehension check (Q33, core)

- **FR-5.1** For each concept new to this session, the diagnostic question (FR-3.2) is a **prediction** component: the "before" prediction, asked before any explanation.
- **FR-5.2** When the student ends the session, Pi shows a new-situation "after" prediction component for the concepts it covered.
- **FR-5.3** The actual result for an "after" prediction comes from one of two places:
  - code the student runs themselves, entered in the prediction component's result field
  - a test run the Builder already did
  
  Pi itself never runs code.
- **FR-5.4** Each prediction component shows the prediction and the actual result side by side once the result is in. The session summary shows the before and after results for each concept.

### FR-6 Session learning record

- **FR-6.1** Pi records, for each concept: the questions asked, the student's answers, the grades, skips and disputes.
- **FR-6.2** Each concept gets a label: not yet shown, shaky or solid. Fixed rules set the label, and "solid" needs correct answers in at least two different exercise types. (Q19)
- **FR-6.3** No live record panel. The session summary component lists the concepts covered, how each went, the before-and-after results and the concepts still weak. (Q23)
- **FR-6.4** Learning state is never carried into a later session.
- **FR-6.5** The session summary reports four of the founding idea's §11 success metrics (gap 5):

| Metric | What the summary reports |
|---|---|
| Prediction | "After" predictions right compared with "before" predictions right |
| Comprehension | Share of teach-back answers graded "got it" or "partly" |
| Interruption cost | How long cards stayed open, how many were skipped, and cards per Claude turn |
| Efficiency | What the session cost (Haiku dollars, Builder calls, videos) next to the before-and-after gain |

  Transfer, retention and independence need more than one session, so they are out of scope (FR-6.4).

### FR-7 Optional video generation (team direction 3)

- **FR-7.1** Video is off by default, and the student turns it on in the Pi window's settings. Pi works fully with videos off, or without the runner installed. (Q8, tech-env §4)
- **FR-7.2** Before turning videos on, Pi checks that everything it needs is present:
  - macOS with a Metal GPU
  - `uv` and Python 3.14
  - a Project-Feynman checkout at `VIDEO_ENGINE_DIR`
  - a logged-in Claude Code
  
  If anything is missing, Pi names it and leaves videos off.
- **FR-7.3** When videos are on, Pi offers a video on a card, and renders it only if the student accepts. (Q8)
- **FR-7.4** Pi makes the video by running `make-video` as a background subprocess with `--agent claude`, using the student's own Claude login. The subprocess never receives `ANTHROPIC_API_KEY`. Jobs run one at a time, without blocking the session, and show a "video rendering" card until the MP4 is ready.
- **FR-7.5** At most 3 videos per session. The window shows how many are left.
- **FR-7.6** If a render fails, Pi replaces the video card with a text explanation of the same concept.

### FR-8 First-run setup (gap 3)

- **FR-8.1** The Anthropic API key is read from a gitignored `.env` file, and a committed `.env.example` lists the variable with no value (tech-env §7). If the key is missing, Pi says so and explains how to add it.
- **FR-8.2** On first launch, a setup screen walks through these steps and shows pass or fail for each:
  1. Choose the repo (FR-1.3).
  2. Install the hooks (FR-1.7).
  3. Check that Claude Code is installed and logged in.
  4. Check that the API key works.
- **FR-8.3** The setup screen can be reopened from settings. A failed step names what is missing and how to fix it.
- **FR-8.4** Video requirements are not part of setup. They are checked only when the student turns videos on (FR-7.2).

---

## 6. Non-functional requirements

### NFR-1 Cost
- **NFR-1.1** Haiku spending is capped at $0.25 per session, about 25 to 50 exercises. Pi stops intervening when it reaches the cap. (Q24)
- **NFR-1.2** At most 15 Builder calls per session. In the MVP these are used for answer-key checks. (Q25)
- **NFR-1.3** At most 3 videos per session (FR-7.5). Video work uses the student's Claude plan, not Pi's API key.
- **NFR-1.4** All caps live in one config file.

### NFR-2 Latency (Q26)
- **NFR-2.1** Explanations and exercises that need no Builder check appear within 15 seconds of the event that triggers them.
- **NFR-2.2** Builder-checked exercises appear within 60 seconds.
- **NFR-2.3** A "preparing" indicator shows while a component is being made.

### NFR-3 Reliability (Q27)
- **NFR-3.1** If a model call fails, Pi retries briefly, then shows a small status notice and resumes on its own once calls work again.
- **NFR-3.2** A failure in the Builder bridge or the video runner never stops the teaching loop.

### NFR-4 Privacy and logging (Q28)
- **NFR-4.1** Pi always logs metadata: event types, timings, token counts and errors.
- **NFR-4.2** Full prompts, model outputs and student answers are logged only when a debug setting is on.
- **NFR-4.3** Logs and learning data are stored in Pi's own app-data folder, never in the student's repo. Pi sends a model only the code snippets and answers it needs.

### NFR-5 Security (Q30: Security extension off; tech-env §7 controls kept as requirements)
- **NFR-5.1** The Electron window runs with `contextIsolation: true`, `sandbox: true` and `nodeIntegration: false`, and a strict Content Security Policy. It loads only the app's own bundle, and navigation and new windows are blocked.
- **NFR-5.2** The preload script exposes only a small typed API.
- **NFR-5.3** The hook server binds to `127.0.0.1`. It accepts only events carrying the random per-session token. Pi makes a new token at every launch and writes it, with the port, to a runtime file in its app-data folder with owner-only permissions (`0600`), so other users on the machine can't read it. Pi's hook script reads the token from that file and sends it with each event. The hook entries in `settings.json` don't change between sessions. *(Reworded 2026-11-25, Application Design clarification 1; it used to say the token is put in the hook command.)*
- **NFR-5.4** Pi's agent has only read tools: `read`, `grep`, `find` and `ls`. Those tools refuse paths outside the repo and secret files (`.env*`, `*.pem`, `id_*`, `.git/`).
- **NFR-5.5** Builder calls run `claude -p` with `--allowedTools Read,Grep,Glob` and `--permission-mode dontAsk`.
- **NFR-5.6** Every hook payload, IPC message, model JSON output and lesson component is checked against a TypeBox schema. Bad input is logged and dropped.
- **NFR-5.7** Subprocesses (`claude`, `make-video`) are started with argument arrays, never a shell string.
- **NFR-5.8** Secrets live in a gitignored `.env`, with a committed `.env.example`. Keys never reach the window.

### NFR-6 Portability and setup
- **NFR-6.1** Pi runs on macOS. The repo is public, so a fresh clone must run without the video runner (FR-7.1).
- **NFR-6.2** Everything runs locally. There is no hosted backend.

### NFR-7 Maintainability
- **NFR-7.1** Contracts sit in front of the parts likely to change:
  - the tutor model, through a `TutorModel` interface
  - the Builder bridge
  - the video runner
  - the component registry
  
  Callers depend on the contract, not the provider.
- **NFR-7.2** Every prompt and every tunable value (caps, timeouts, the concept list) lives in one config module.

### NFR-8 Testing (Q31: PBT partial)
- **NFR-8.1** Tests:
  - Vitest for unit and integration tests
  - Testing Library for component renderers
  - one Playwright Electron end-to-end test of the demo flow
- **NFR-8.2** Unit tests never call a real model. They use a fake `TutorModel` and recorded hook fixtures. An opt-in live suite checks Haiku's output against the schemas.
- **NFR-8.3** Property-based testing uses fast-check in partial mode. Rules PBT-02, PBT-03, PBT-07, PBT-08 and PBT-09 are blocking, and the rest are advisory. Targets:
  - schema round-trips
  - lesson component validation
  - mastery-label rules
  - cap accounting

---

## 7. Demo and acceptance criteria

- **AC-1 Demo app:** a real-time WebSocket chat server in TypeScript, built live with Claude Code. (Q32)
- **AC-2 Live demo only:** a teammate uses Claude Code in real time, and the video shows Pi's real responses. No replay mode is built. (Q35)
- **AC-3 End to end:** in one session on the demo app, every MVP component type that responds to the student appears at least once. The before-and-after check runs end to end, nothing crashes, and the automated tests pass. (Q34)
- **AC-4 Answer-key quality:** teammates hand-review 20 exercises generated from recorded hook fixtures. At least 90% have a correct answer key for the actual code. (Q34)
- **AC-5 Works without videos:** a clean machine without the video runner runs the full demo flow, and turning videos on shows what is missing.
- **AC-6 Clean install:** following the README on a fresh clone, a teammate gets through the setup screen (FR-8), and removing Pi's hooks leaves `~/.claude/settings.json` exactly as it was before installing.

## 8. Out of scope

- **Stretch goals:**
  - student-drawn diagrams or a pseudocode editor
  - Builder calls for explanations or concept identification
  - card-scoped student questions to Pi
- **Not built:**
  - blocking or gating Claude Code
  - a live record panel
  - OS notifications
  - "ask me later" or snooze
  - Builder rechecks of disputed grades
  - a demo replay mode
  - languages other than JS/TS
- **Still v2 or later (founding idea §10):**
  - difficulty scaling
  - remediation videos
  - persistent profiles across sessions
  - a mastery dashboard
  - caching
  - configurable interruption modes
  - other coding agents

## 9. Extension configuration

| Extension | Decision | Effect |
|---|---|---|
| Security Baseline | No (Q30) | Rules not loaded. tech-env §7 controls kept as NFR-5 |
| Resiliency Baseline | No (Q29) | Rules not loaded |
| Property-Based Testing | Partial (Q31) | Rules loaded. PBT-02, 03, 07, 08 and 09 are blocking; the rest are advisory |

## 10. Traceability

| Question | Requirement |
|---|---|
| Q1, Q2 | §4 Users |
| Q3 | FR-2.7, §8 |
| Q4, Q7, Q15–Q18, Q22 | FR-3 |
| Q5, Q12, Q20, Q21 | FR-4 |
| Q6 | FR-2.5 |
| Q8 | FR-7 |
| Q9 | §3, FR-2.7 |
| Q10, Q11, Q13, Q14 | FR-1 |
| Q19, Q23 | FR-6 |
| Q24–Q28 | NFR-1 to NFR-4 |
| Q29–Q31 | §9, NFR-5, NFR-8 |
| Q32, Q34, Q35 | §7 |
| Q33 | FR-5 |
| Team directions 1–3 | §1, FR-2, FR-7 |
| Team direction: Pi is a teaching assistant agent | §1, §2, FR-0 |
| Gap answers (2026-11-23): before-and-after component, hook install, first-run setup, metrics, README | FR-2.7, FR-5, FR-1.7, FR-8, FR-6.5, §3, AC-6 |
