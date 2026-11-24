# User Stories: Anti-Vibe Coding (Pi, the learning sidecar)

**Stage:** INCEPTION - User Stories (Part 2: Generation)
**Sources:** requirements.md (approved 2026-11-23), personas.md, story-generation-plan.md (approved 2026-11-24), story-generation-clarification-questions.md

## Decisions

The rules in this file come from the approved story plan:

| Topic | Rule |
|---|---|
| Organization (Q4 D) | Epics E1 to E8 follow the student's journey. The stories inside each epic are cut by feature. E9 groups the optional video stories |
| Size (Q5 B, Q7 A, follow-up 4 A) | 33 core stories and 3 video stories (video doesn't count toward the 25 to 35 range). Each story is labeled **S**, **M** or **L**. **L** means the story crosses several parts of Pi (window, agent, schemas and record) or adds a new contract |
| Component types (Q6 B) | US-13 covers the shared component system. US-18 to US-21 cover the four types the student responds to. Display-only types are criteria in US-14, US-15, US-31 and US-V2 |
| Criteria format (Q8 C) | **Given/When/Then** for behavior the student triggers. **Checklist** bullets for rules and limits |
| Labels (Q9 C, follow-up 3 A) | Each criterion is **[test]** (decided by an automated test) or **[review]** (a teammate judges it against the rubric below). [review] appears only on lesson-content and grading stories, including how relevant Pi's choices are |
| Failure paths (Q10 A) | Every story where something can fail has at least one criterion marked ⚠ |
| Actors (Q12 C, Q13 B, follow-up 2 A) | Student stories by default. "As Pi…" stories (US-11, US-12, US-24) cover the agent's decisions. Limits on the agent are student trust and cost stories (US-09, US-27). "As a Pi developer…" stories cover only logging and the config module (US-32, US-33) |
| Traceability (Q11 A) | Each story lists its requirement IDs. The reverse table is at the end |

**[review] rubric.** A teammate marks a [review] criterion as passing when all of these hold for the sample:
1. The content is about the code actually on screen.
2. It is correct for that code.
3. Every technical term is defined the first time it appears.
4. A student at the persona's level could act on it.

Unless a criterion names another sample, the sample is 10 outputs from recorded hook fixtures.

**Definition of done, for every story:**
- NFR-7.1: the code goes through the right contract (`TutorModel`, Builder bridge, video runner, component registry).
- NFR-8.1 and NFR-8.2: it has Vitest tests, and Testing Library tests for renderers. Tests use the fake `TutorModel` and recorded fixtures.
- NFR-8.3: the PBT targets it touches have fast-check properties.
- AC-3: the automated test suite passes.

---

## Journeys

### J1: The MVP demo session (Riley, built live; AC-1, AC-2, AC-3)

| # | Step | Stories |
|---|---|---|
| 1 | Riley launches Pi. Setup is already green from an earlier run. They confirm the `chat-server` repo | US-02, US-03 |
| 2 | Riley asks Claude Code for a WebSocket server that broadcasts to every client. The session starts | US-06, US-08 |
| 3 | Claude edits `server.ts`. Pi decides *event handlers* is worth teaching and shows a short explanation live, with a diagram of connection, message and broadcast | US-11, US-14, US-15, US-16 |
| 4 | *Broadcast* is new to this session, so Pi asks a "before" prediction first: what do B and C see when A sends "hi"? | US-12, US-21 |
| 5 | Claude's turn ends. Pi shows a multiple-choice card that the Builder has checked: what happens to `clients` when a socket closes? | US-24, US-18 |
| 6 | Riley gets it wrong and picks **Drill down**: two follow-ups, then a retest with a new example | US-25 |
| 7 | Next turn: a written teach-back on broadcast, graded with a checklist. Then a quick-time event, which Riley lets time out (counted as skipped) | US-19, US-20, US-22 |
| 8 | Riley disputes one grade | US-26 |
| 9 | Riley presses **End session**. An "after" prediction in a new situation, then the summary with before-and-after results and the four metrics | US-29, US-30, US-31 |

**What changed from founding idea §7:**
- Pi doesn't ask the Builder for explanations. It asks only to check answer keys (a stretch goal now).
- There is no pseudocode design exercise. Diagrams are display-only.
- There is no "ask me later." Nothing carries over to a future session.
- Pi starts every exchange. The student never types a free question.

### J2: Setup on a fresh clone without the video runner (Sam; AC-5, AC-6)

| # | Step | Stories |
|---|---|---|
| 1 | Sam clones the public repo on a Mac without Project Feynman and follows the README | US-01 |
| 2 | Pi launches without a key in `.env`. It says so and explains how to add one. Sam adds the key and relaunches | US-01 |
| 3 | The setup screen walks Sam through four steps, each with pass or fail | US-02 |
| 4 | Sam picks the demo repo and sees the exact hook entries Pi will add. They confirm, and their own hooks stay as they were | US-03, US-04 |
| 5 | Sam turns videos on. Pi names the missing requirement and leaves videos off. The demo runs fully without video | US-V1 |
| 6 | Sam closes Pi and runs Claude Code. The hooks exit at once, with no error and no delay | US-07 |
| 7 | Sam reopens Pi and presses **Remove Pi hooks**. `~/.claude/settings.json` matches the backup exactly | US-05 |

---

## E1: Set up Pi

### US-01 Add my API key
**As** Sam, the demo teammate, **I want** Pi to read its API key from a local `.env` file and tell me plainly when the key is missing, **so that** I can set it up from the README without leaking the key.
**Size:** S · **Personas:** P3 (main), P1, P2 · **Covers:** FR-8.1, NFR-5.8, NFR-6.1

**Acceptance criteria**
- [test] The repo has a committed `.env.example` that lists `ANTHROPIC_API_KEY` with no value. `.env` is in `.gitignore`.
- [test] The key is read in the main process only. It never appears in a renderer message, the preload API or the window.
- ⚠ [test] **Given** no `.env` or an empty key, **when** Pi launches, **then** it shows a message naming the missing variable and how to add it, and it doesn't start teaching.

### US-02 Get through first-run setup
**As** Sam, **I want** a setup screen that checks each requirement and shows pass or fail, **so that** I know Pi is ready before the demo starts.
**Size:** M · **Personas:** P3 (main), P1, P2 · **Covers:** FR-8.2, FR-8.3, FR-8.4, AC-6

**Acceptance criteria**
- [test] **Given** a first launch, **when** the window opens, **then** the setup screen shows four steps in order: choose the repo, install the hooks, check Claude Code is installed and logged in, check the API key works.
- [test] **Given** all four steps pass, **when** the student continues, **then** the lesson feed opens.
- ⚠ [test] **Given** Claude Code isn't installed or isn't logged in, **when** that step runs, **then** it shows fail, names what is missing and how to fix it, and lets the student retry.
- ⚠ [test] **Given** the API key is rejected, **when** that step runs, **then** it shows fail with a fix hint, and the key value isn't displayed.
- [test] The setup screen can be reopened from settings.
- [test] The setup screen doesn't check video requirements.

### US-03 Choose the repo Pi watches
**As** Riley, **I want** to pick the one project folder Pi pays attention to, **so that** Pi teaches about my project and ignores other Claude Code sessions.
**Size:** S · **Personas:** P1 (main), P2, P3 · **Covers:** FR-1.3, NFR-6.2

**Acceptance criteria**
- [test] **Given** the setup screen, **when** Riley picks a folder, **then** Pi stores that path in its own app-data folder and shows it in the window.
- [test] **Given** a chosen repo, **when** a hook event arrives with a `cwd` outside it, **then** Pi ignores the event and logs only that it was ignored.
- ⚠ [test] **Given** the chosen folder no longer exists, **when** Pi launches, **then** the repo step shows fail and asks for a new folder.
- [test] Nothing is sent off the machine except model calls (NFR-6.2).

### US-04 Install Pi's hooks with my consent
**As** Sam, who already has hooks of their own, **I want** to see exactly what Pi adds to my Claude Code settings before it changes anything, **so that** my setup isn't broken.
**Size:** M · **Personas:** P3 (main), P1, P2 · **Covers:** FR-1.4, FR-1.7, NFR-5.3

**Acceptance criteria**
- [test] **Given** the hooks step, **when** it opens, **then** Pi shows the exact `PostToolUse`, `Stop` and `UserPromptSubmit` entries it will add to the user-level `~/.claude/settings.json`, and changes nothing until the student confirms.
- [test] **Given** existing settings and hooks, **when** Sam confirms, **then** Pi adds only its own entries, and every other key and hook is unchanged. A property-based test merges random settings files.
- [test] No file in the student's repo is created or changed.
- [test] The hook command carries a per-session token (US-06).
- ⚠ [test] **Given** `settings.json` isn't valid JSON, **when** Sam confirms, **then** Pi doesn't write anything and tells Sam the file needs fixing first.
- ⚠ [test] **Given** Sam cancels, **when** the step closes, **then** the file is byte-for-byte unchanged, and the step shows "not installed."

### US-05 Remove Pi's hooks cleanly
**As** Sam, **I want** one button that removes only Pi's hooks, **so that** my Claude Code settings go back to exactly how they were.
**Size:** S · **Personas:** P3 (main), P2 · **Covers:** FR-1.7, AC-6

**Acceptance criteria**
- [test] **Given** Pi's hooks are installed, **when** Sam presses **Remove Pi hooks**, **then** only Pi's entries are deleted.
- [test] After install and then removal, `settings.json` is identical to the file before install. A property-based round-trip test checks this on random files.
- ⚠ [test] **Given** Sam edited `settings.json` after installing, **when** they remove the hooks, **then** their edits stay, and only Pi's entries go.
- ⚠ [test] **Given** Pi's entries are already gone, **when** Sam presses the button, **then** nothing changes, and Pi says no Pi hooks were found.

---

## E2: Pair with Claude Code

### US-06 Pi follows what Claude Code does
**As** Riley, **I want** Pi to follow my Claude Code session as it happens, and only my session, **so that** its lessons match what Claude just did.
**Size:** M · **Personas:** P1 (main), P2, P3 · **Covers:** FR-1.5, FR-0.2, NFR-5.3, NFR-5.6

**Acceptance criteria**
- [test] The hook server listens on `127.0.0.1` only.
- [test] **Given** a `PostToolUse`, `Stop` or `UserPromptSubmit` event with this session's token, **when** it arrives, **then** Pi passes it to the agent as a trigger.
- ⚠ [test] **Given** an event with a missing or wrong token, **when** it arrives, **then** Pi drops it and logs the rejection.
- ⚠ [test] **Given** a payload that fails its TypeBox schema, **when** it arrives, **then** Pi drops it and logs it. A property-based test sends random payloads.
- [test] Pi learns Claude's decisions and test results only from the end-of-turn message, the transcript and test commands Claude already ran. It runs nothing itself.

### US-07 Pi never slows or blocks Claude Code
**As** Devin, **I want** Claude Code to run at full speed whether Pi is open, busy or closed, **so that** learning never costs me building time.
**Size:** S · **Personas:** P2 (main), P1, P3 · **Covers:** FR-1.6, FR-1.7, NFR-3.2

**Acceptance criteria**
- [test] Hooks return at once. They never wait for Pi's agent, a model call or the Builder.
- ⚠ [test] **Given** Pi isn't running, **when** a hook fires, **then** it exits with status 0 and no output, within 200 ms.
- [test] Every exercise is about code Claude has already written. Pi never asks Claude Code to pause.
- ⚠ [test] **Given** the Builder bridge or video runner fails, **when** the next event arrives, **then** the teaching loop keeps working.

### US-08 My session lasts until I close Pi
**As** Riley, **I want** Pi to keep track of what I've learned for as long as Pi is open, even if I restart Claude Code or `/clear`, **so that** I don't lose progress mid-session.
**Size:** S · **Personas:** P1 (main), P2 · **Covers:** FR-1.1, FR-0.5, FR-6.4

**Acceptance criteria**
- [test] **Given** an open Pi session, **when** Claude Code restarts or runs `/clear`, **then** the learning record and feed stay as they were.
- [test] **Given** Pi is closed and reopened, **when** the new session starts, **then** the learning record starts empty. No learning state is carried over.
- [test] The agent's only working state is the session learning record.

### US-09 Pi can't touch my repo or my secrets
**As** Devin, **I want** Pi to be able only to read my project, and never my secrets, **so that** I can trust it beside a real project.
**Size:** M · **Personas:** P2 (main), P1, P3 · **Covers:** FR-0.3, NFR-5.4, NFR-5.5, NFR-5.7, NFR-4.3

**Acceptance criteria**
- [test] The agent's only tools are read-only `read`, `grep`, `find` and `ls`, `ask_builder`, show lesson component and, with videos on, offer video. No tool writes, runs code or sends anything to Claude Code's live session.
- ⚠ [test] **Given** a path outside the repo, or a secret file (`.env*`, `*.pem`, `id_*`, `.git/`), **when** a code tool is called with it, **then** the tool refuses and logs the refusal. A property-based test generates random paths, including `..` and symlinks.
- [test] Builder calls run `claude -p` with `--allowedTools Read,Grep,Glob` and `--permission-mode dontAsk`.
- [test] Subprocesses are started with argument arrays, never a shell string.
- [test] Pi sends a model only the code snippets and answers that call needs.
- [test] After a full demo session, `git status` in the student's repo shows no change made by Pi.

### US-10 The Pi window shows only lessons
**As** Devin, **I want** the Pi window to show only Pi's own lesson cards, **so that** nothing a model writes can run on my machine.
**Size:** S · **Personas:** P2 (main), P1 · **Covers:** NFR-5.1, NFR-5.2, FR-2.3

**Acceptance criteria**
- [test] The window runs with `contextIsolation: true`, `sandbox: true` and `nodeIntegration: false`, and a strict Content Security Policy.
- [test] The window loads only the app's own bundle. Navigation and new windows are blocked.
- [test] The preload script exposes only the typed API in its contract.
- ⚠ [test] **Given** model output that contains HTML or a script, **when** it is rendered, **then** it shows as plain text and doesn't run.

---

## E3: Learn while Claude builds

### US-11 As Pi, decide whether something is worth teaching
**As** Pi, **I want** to decide on each trigger whether a change is worth teaching right now, **so that** the student learns the important ideas without being flooded.
**Size:** M · **Personas:** Pi (agent) · **Covers:** FR-0.1, FR-0.2, FR-0.4, FR-3.1

**Acceptance criteria**
- [test] Pi's agent runs on the pi agent harness with Haiku 5.5, through the `TutorModel` contract.
- [test] Triggers are: a hook event, a student response, or **End session**.
- [test] Haiku tags each change only with concepts from the fixed list (about 20) in the config module. A tag that isn't on the list is dropped.
- [test] **Given** no concept on the list applies, **when** the run finishes, **then** Pi shows nothing and logs why.
- [test] A run ends when it shows one component, decides nothing is worth teaching, reaches its step limit, or reaches a session cap.
- ⚠ [test] **Given** the agent reaches its step limit, **when** the run stops, **then** nothing half-finished is shown, and the reason is logged.
- [review] For 10 recorded fixture sessions, the concepts Pi chose to teach are the ones a teammate judges most worth teaching for that change.

### US-12 As Pi, choose the next lesson for a concept
**As** Pi, **I want** to pick the kind of lesson from what the student has shown this session, **so that** I measure first what they know and help first when they've missed.
**Size:** M · **Personas:** Pi (agent) · **Covers:** FR-3.2, FR-5.1, FR-2.1

**Acceptance criteria**
- [test] **Given** a concept new to this session, **when** Pi teaches it, **then** the first component is a "before" prediction, shown before any explanation of that concept.
- [test] **Given** a concept the student missed this session, **when** Pi returns to it, **then** it explains first.
- [test] Pi picks only registered component types.
- [review] The component type Pi picks suits the concept and the moment, for example a diagram for a message flow and a quick-time event for a fact that can be checked quickly.

### US-13 See every lesson as a card in one feed
**As** Riley, **I want** everything Pi shows me to appear as cards in a single feed that I answer with each card's own controls, **so that** lessons are easy to follow and nothing odd can appear.
**Size:** L (crosses the window, agent, schemas and record, and adds the component registry contract) · **Personas:** P1 (main), P2, P3 · **Covers:** FR-2.1, FR-2.2, FR-2.3, FR-2.4, FR-2.5, FR-2.6, NFR-5.6, NFR-7.1

**Acceptance criteria**
- [test] Each component type is registered with a TypeBox field schema, a response schema (or none), a renderer and an optional grader.
- [test] The agent can emit only registered types.
- [test] Cards appear in order as a chat-style feed. The student answers only through each card's controls. There is no free-text box for questions to Pi.
- ⚠ [test] **Given** a component that fails its schema, **when** it is generated, **then** Pi regenerates it once. If it fails again, Pi drops and logs it, and nothing is shown. A property-based test generates invalid components.
- [test] Schemas round-trip: encode then decode gives back the same component. This is a property-based test.
- [test] Adding a test-only component type to the registry needs no change to the agent loop, feed, grading flow or session record.

### US-14 Get short explanations while Claude edits
**As** Riley, **I want** short explanations of the code Claude is writing, tied to the exact lines, **so that** I understand it as it happens.
**Size:** M · **Personas:** P1 (main), P2 · **Covers:** FR-2.7 (Explanation), FR-3.3, §4 Users

**Acceptance criteria**
- [test] **Given** Claude edits a significant file, **when** Pi decides to teach, **then** an explanation card appears during the turn, without waiting for the turn to end.
- [test] Each explanation has text and a code snippet with the file path and line references.
- [review] Each technical term is defined the first time it appears in the session.
- [review] The explanation is correct for the code shown and is pitched at the intermediate level.
- ⚠ [test] **Given** the file referenced has been deleted, **when** the card renders, **then** it shows the snippet it was given and notes that the file changed.

### US-15 See diagrams and small charts
**As** Riley, **I want** diagrams of how the parts connect and small charts where data helps, **so that** I can see structure, not only read about it.
**Size:** S · **Personas:** P1 (main), P2 · **Covers:** FR-2.7 (System design graph, Data graphic)

**Acceptance criteria**
- [test] A system design graph card draws Mermaid text in the window. It is display-only.
- [test] A data graphic card draws a bar chart, line chart or table from data written in the component. It can't load anything from the network or run code.
- ⚠ [test] **Given** Mermaid text that doesn't parse, **when** the card renders, **then** it shows the text as a plain code block with a note, and no error.
- [review] The diagram matches the structure of the actual code.

### US-16 Get cards at a pace that doesn't break my flow
**As** Devin, **I want** exercises to wait until Claude's turn ends, with no more than two per turn, and no pop-ups, **so that** I can keep steering Claude.
**Size:** S · **Personas:** P2 (main), P1 · **Covers:** FR-3.3, FR-3.4

**Acceptance criteria**
- [test] Exercises appear only after a `Stop` event, at most two per Claude turn. The limit is in the config module.
- [test] Explanations may appear live during the turn.
- [test] A new card shows a badge or highlight inside the Pi window only. No OS notification, and no stolen focus.
- [test] The window is a normal window that the student can move and resize.
- ⚠ [test] **Given** a third exercise is ready in the same turn, **when** the limit is reached, **then** it isn't shown, and the reason is logged.

### US-17 Know Pi is working on a card
**As** Sam, during the live demo, **I want** to see when Pi is preparing a card and get it within a set time, **so that** a quiet Pi doesn't look broken.
**Size:** S · **Personas:** P3 (main), P1, P2 · **Covers:** NFR-2.1, NFR-2.2, NFR-2.3

**Acceptance criteria**
- [test] A "preparing" indicator shows while a component is being made.
- [test] With the fake `TutorModel`, cards that need no Builder check appear within 15 seconds of their trigger, and Builder-checked cards within 60 seconds. The timings are logged so they can be checked in the live demo.
- ⚠ [test] **Given** a card isn't ready within its time limit, **when** the limit passes, **then** the indicator clears, and the card is dropped and logged.

---

## E4: Answer exercises

### US-18 Answer a multiple-choice question
**As** Riley, **I want** to answer multiple-choice questions about the code Claude just wrote, **so that** I find out quickly whether I understood it.
**Size:** S · **Personas:** P1 (main), P2 (main) · **Covers:** FR-2.7 (Multiple choice), FR-4.1, FR-1.6

**Acceptance criteria**
- [test] **Given** a multiple-choice card, **when** Riley picks an option, **then** it is graded right or wrong, and the result is saved to the learning record.
- [test] **Given** an architecture question ("which approach would you have picked?"), **when** Devin answers, **then** the card shows what Claude actually built, compared with Devin's pick.
- [review] The answer key is correct for the actual code, and the wrong options are plausible.
- ⚠ [test] **Given** the student responds twice quickly, **when** the second response arrives, **then** only the first is graded.

### US-19 Explain it back in my own words
**As** Devin, **I want** to explain a piece of code in my own words and see exactly which points I hit and missed, **so that** I know whether I really understand it.
**Size:** M · **Personas:** P2 (main), P1 · **Covers:** FR-2.7 (Written response), FR-4.2

**Acceptance criteria**
- [test] The checklist is drafted from the actual code, in the same model call that writes the question.
- [test] **Given** Devin submits an answer, **when** it is graded, **then** the verdict is "got it", "partly" or "not yet", and the card lists the checklist points hit and missed.
- [test] **Given** "partly", **when** the verdict shows, **then** Pi offers one targeted follow-up. **Given** "not yet", Pi starts a drill-down (US-25).
- [review] For 10 sample answers, the verdict matches a teammate's own grade against the same checklist.
- [review] The checklist points cover what matters in the code and no more.
- ⚠ [test] **Given** the grading call fails, **when** the student submits, **then** their answer is kept, and the card says grading will retry (US-28).

### US-20 Answer a quick-time event
**As** Riley, **I want** quick questions with a countdown, **so that** practicing feels quick and tests what I know without looking it up.
**Size:** S · **Personas:** P1 (main), P2, P3 · **Covers:** FR-2.7 (Quick-time event), FR-4.1, FR-3.5

**Acceptance criteria**
- [test] **Given** a quick-time card, **when** Riley picks an option before the countdown ends, **then** it is graded right or wrong.
- [test] **Given** the countdown ends with no answer, **when** time runs out, **then** it is recorded as skipped, not wrong.
- [test] The countdown length is in the config module.
- [review] The question can fairly be answered within the countdown.

### US-21 Predict what the code will do before Pi explains
**As** Riley, **I want** to predict what the code will do before Pi explains a new concept, **so that** I and Pi can see what I knew before this session.
**Size:** M · **Personas:** P1 (main), P2 · **Covers:** FR-2.7 (Prediction), FR-5.1, FR-5.4

**Acceptance criteria**
- [test] **Given** a "before" prediction card, **when** Riley picks an option or writes a short prediction, **then** the prediction is saved for that concept as "before".
- [test] **Given** the actual result is known (from a test Claude already ran, or entered by the student), **when** it is added, **then** the card shows the prediction and the actual result side by side.
- [test] Pi itself never runs code to get the actual result.
- [review] The predicted situation is concrete and comes from the actual code.
- ⚠ [test] **Given** no actual result arrives before session end, **when** the summary is built, **then** that "before" prediction shows as "no result" and isn't counted as right or wrong.

### US-22 Skip an exercise without penalty
**As** Riley, **I want** to skip any exercise, **so that** I'm never stuck or punished when I'm not ready.
**Size:** S · **Personas:** P1 (main), P2 (main) · **Covers:** FR-3.5, FR-6.1

**Acceptance criteria**
- [test] **Given** any exercise card, **when** Riley presses **Skip**, **then** it is recorded as "skipped", not wrong.
- [test] A skipped question isn't asked again this session.
- [test] Skips count toward the interruption-cost metric (US-31).

### US-23 Answer a question about code that changed
**As** Devin, **I want** a question to stay as it was asked even if Claude changes the code meanwhile, **so that** I'm graded fairly.
**Size:** S · **Personas:** P2 (main), P1 · **Covers:** FR-3.6

**Acceptance criteria**
- [test] **Given** a question on screen, **when** the code it refers to changes, **then** the question stays as it was, and the card notes that the code has since changed.
- [test] Grading uses the code as it was when the question was asked.

### US-24 As Pi, check answer keys with the Builder before asking
**As** Pi, **I want** the Builder to confirm the answer key of any question whose answer depends on how the code runs, **so that** I never mark a student wrong with a wrong key.
**Size:** M · **Personas:** Pi (agent) · **Covers:** FR-4.5, FR-0.3, NFR-1.2, NFR-2.2, AC-4

**Acceptance criteria**
- [test] Haiku flags each question whose answer depends on how the code runs. Flagged questions are held until checked.
- [test] **Given** a flagged question, **when** the Builder (read-only `claude -p`) agrees with the key, **then** the question is shown.
- ⚠ [test] **Given** the Builder disagrees, times out, or the cap of 15 Builder calls is reached, **when** that happens, **then** the question is dropped and logged, and the student sees nothing broken.
- [test] Each Builder call counts toward the session cap (US-27).
- [review] AC-4: teammates hand-review 20 exercises generated from recorded hook fixtures. At least 90% have a correct answer key for the actual code.

---

## E5: Recover from wrong answers

### US-25 Choose to see the answer or drill down
**As** Riley, **I want** to choose between seeing the answer and working through it step by step after I get something wrong, **so that** I can recover in the way that suits me.
**Size:** M · **Personas:** P1 (main), P2 · **Covers:** FR-4.3, FR-4.2

**Acceptance criteria**
- [test] **Given** a wrong answer or a "not yet" verdict, **when** the result shows, **then** the card offers **Show me the answer** and **Drill down**.
- [test] **Given** Riley picks **Show me the answer**, **when** they press it, **then** the answer and why it is right are shown.
- [test] **Given** Riley picks **Drill down**, **when** they press it, **then** Pi gives at most two focused follow-ups and one retest with a new example, then reveals the answer either way.
- [review] The follow-ups break the idea into smaller steps, and the retest really is a new example.
- ⚠ [test] **Given** a session cap is reached mid-drill-down, **when** the next step is due, **then** Pi reveals the answer and ends the drill-down.

### US-26 Dispute a grade
**As** Devin, **I want** to mark a grade I think is wrong as disputed, **so that** the summary shows it rather than counting it silently.
**Size:** S · **Personas:** P2 (main), P1 · **Covers:** FR-4.4

**Acceptance criteria**
- [test] **Given** any graded card, **when** Devin presses **Dispute**, **then** the grade is marked "disputed" in the learning record.
- [test] Nothing is rechecked. The Builder isn't called.
- [test] The summary lists disputed grades.

---

## E6: Keep costs and failures in check

### US-27 Stay within the session's cost limits
**As** Devin, **I want** Pi to stop at fixed limits on spending, Builder calls and videos, **so that** one session never costs much.
**Size:** S · **Personas:** P2 (main), P1, P3 · **Covers:** NFR-1.1, NFR-1.2, NFR-1.3, FR-0.4

**Acceptance criteria**
- Caps, all read from the config module:
  - [test] Haiku: $0.25 per session.
  - [test] Builder calls: 15 per session.
  - [test] Videos: 3 per session.
- [test] Cap accounting never goes over a cap and never counts one call twice. This is a property-based test.
- ⚠ [test] **Given** the Haiku cap is reached, **when** the next trigger arrives, **then** Pi stops intervening and shows a notice once. **End session** and the summary still work, using no more Haiku.
- [test] Video work uses the student's Claude plan, not Pi's API key.

### US-28 Keep going when a model call fails
**As** Sam, **I want** Pi to recover on its own when a model call fails, **so that** a network glitch doesn't end the demo.
**Size:** S · **Personas:** P3 (main), P1, P2 · **Covers:** NFR-3.1, NFR-3.2

**Acceptance criteria**
- ⚠ [test] **Given** a model call fails, **when** it is retried briefly and still fails, **then** a small status notice appears in the window.
- [test] **Given** calls work again, **when** the next call succeeds, **then** the notice clears, and teaching resumes without the student doing anything.
- [test] A failure in the Builder bridge or video runner never stops the teaching loop.

---

## E7: End the session

### US-29 Make my "after" predictions
**As** Riley, **I want** to predict what new code will do for each concept Pi taught when I end the session, **so that** I can see whether I improved.
**Size:** M · **Personas:** P1 (main), P2 · **Covers:** FR-1.2, FR-5.2, FR-5.3, FR-5.4

**Acceptance criteria**
- [test] **Given** an open session, **when** Riley presses **End session**, **then** Pi shows a new-situation "after" prediction card for each concept it covered.
- [test] The actual result comes from code the student runs themselves and enters in the card's result field, or from a test run the Builder already did. Pi never runs code.
- [test] Each card shows prediction and actual result side by side once the result is in.
- [review] The "after" situation is new (not the "before" one reworded) and tests the same concept.
- ⚠ [test] **Given** Riley skips an "after" card, **when** the summary is built, **then** that concept shows "after: skipped."

### US-30 Have my learning recorded fairly
**As** Riley, **I want** Pi to record how I did on each concept with clear rules, **so that** my summary is fair and explainable.
**Size:** S · **Personas:** P1 (main), P2 · **Covers:** FR-6.1, FR-6.2

**Acceptance criteria**
- [test] For each concept, the record holds the questions asked, the answers, the grades, the skips and the disputes.
- Labels, set by fixed rules and not by the model:
  - [test] The labels are "not yet shown", "shaky" and "solid".
  - [test] "Solid" needs correct answers in at least two different exercise types.
- [test] The label rules are checked with property-based tests. For example: a skip never raises or lowers a label, and "solid" is impossible with fewer than two types.

### US-31 See my session summary
**As** Riley, **I want** a summary at the end that shows what I learned, how my predictions changed and what the session cost, **so that** I know what to work on next.
**Size:** M · **Personas:** P1 (main), P2 (main), P3 · **Covers:** FR-6.3, FR-6.5, FR-2.7 (Session summary), FR-1.2

**Acceptance criteria**
- [test] **Given** the "after" cards are done or skipped, **when** the summary appears, **then** it lists the concepts covered, each one's label and its before-and-after results, and the concepts still weak.
- The summary reports four metrics:
  - [test] Prediction: "after" predictions right compared with "before" predictions right.
  - [test] Comprehension: the share of teach-backs graded "got it" or "partly".
  - [test] Interruption cost: how long cards stayed open, how many were skipped, and cards per Claude turn.
  - [test] Efficiency: Haiku dollars, Builder calls and videos, next to the before-and-after gain.
- [test] There is no live record panel. The summary is the only view of the record.
- ⚠ [test] **Given** a session with no concepts taught, **when** **End session** is pressed, **then** the summary says so, and all metrics show zero, with no errors.

---

## E8: Behind the scenes (Pi developer stories)

### US-32 Log what Pi did, without logging what the student wrote
**As a** Pi developer, **I want** metadata logs of every agent run, with full content only in debug mode, **so that** we can debug and measure Pi without collecting student data by default.
**Size:** S · **Personas:** Pi developer · **Covers:** FR-0.6, NFR-4.1, NFR-4.2, NFR-4.3

**Acceptance criteria**
- [test] Every agent run logs its trigger, tool calls, the component shown or the reason none was shown, timings and tokens.
- [test] Full prompts, model outputs and student answers are logged only when the debug setting is on.
- [test] Logs are written to Pi's own app-data folder, never to the student's repo.
- ⚠ [test] **Given** the log can't be written, **when** a run ends, **then** teaching continues, and the failure is reported once in the window.

### US-33 Keep every prompt and limit in one config module
**As a** Pi developer, **I want** every prompt and every tunable value in one config module, **so that** we can tune Pi without hunting through the code.
**Size:** S · **Personas:** Pi developer · **Covers:** NFR-7.2, NFR-1.4

**Acceptance criteria**
- [test] The config module holds the caps, timeouts, step limit, exercise limit per turn, countdown length, concept list and every prompt.
- [test] A test fails if a prompt string or cap value appears outside the config module.
- ⚠ [test] **Given** a config value is invalid (for example a negative cap), **when** Pi starts, **then** it refuses to start and names the value.

---

## E9: Videos (Optional feature)

Video is off by default and isn't needed for the demo (AC-5). These stories don't count toward the 25 to 35 core range.

### US-V1 Turn on videos only when my machine can make them · *Optional feature*
**As** Sam, **I want** Pi to check my machine before turning videos on and tell me what's missing, **so that** I know why videos are off.
**Size:** M · **Personas:** P3 (main), P1 · **Covers:** FR-7.1, FR-7.2, FR-8.4, AC-5, NFR-6.1

**Acceptance criteria**
- [test] Videos are off by default and are turned on only in the window's settings.
- [test] **Given** Sam turns videos on, **when** Pi checks, **then** it checks for macOS with a Metal GPU, `uv` and Python 3.14, a Project-Feynman checkout at `VIDEO_ENGINE_DIR`, and a logged-in Claude Code.
- ⚠ [test] **Given** any of these is missing, **when** the check finishes, **then** Pi names each missing item and leaves videos off.
- [test] With videos off, or the runner not installed, the full demo flow works (AC-5), and the offer-video tool isn't available to the agent.

### US-V2 Watch a short video about a concept when I want one · *Optional feature*
**As** Riley, **I want** Pi to offer a short video on a concept and make it only if I accept, **so that** I can see an idea animated without slowing my session.
**Size:** L (adds the video runner contract and a background subprocess) · **Personas:** P1 (main), P3 · **Covers:** FR-7.3, FR-7.4, FR-2.7 (Video), FR-0.3, NFR-5.7

**Acceptance criteria**
- [test] **Given** videos are on, **when** Pi offers a video on a card, **then** nothing renders until Riley accepts.
- [test] **Given** Riley accepts, **when** the job starts, **then** a "video rendering" card shows until the MP4 is ready, and then a video card plays it.
- [test] `make-video` runs as a background subprocess with `--agent claude` and an argument array. Its environment doesn't include `ANTHROPIC_API_KEY`.
- [test] Jobs run one at a time, and the session continues while a video renders.
- [review] The video explains the concept it was offered for, correctly.

### US-V3 Know my video limit and still learn if a video fails · *Optional feature*
**As** Riley, **I want** to see how many videos I have left and get a text explanation if one fails, **so that** video problems never cost me the lesson.
**Size:** S · **Personas:** P1 (main), P3 · **Covers:** FR-7.5, FR-7.6, NFR-1.3, NFR-3.2

**Acceptance criteria**
- [test] At most 3 videos per session. The window shows how many are left.
- [test] **Given** the limit is reached, **when** the agent runs, **then** no more videos are offered.
- ⚠ [test] **Given** a render fails, **when** the job ends, **then** the video card is replaced with a text explanation of the same concept, and the session continues.

---

## Not in this release

These items appear in requirements §8 and aren't written as stories (Q16 B):
- **Stretch goals:**
  - student-drawn diagrams or a pseudocode editor
  - Builder calls for explanations or concept identification
  - card-scoped questions from the student to Pi
- **Not built:**
  - blocking or gating Claude Code
  - a live record panel
  - OS notifications
  - "ask me later" or snooze
  - Builder rechecks of disputed grades
  - a demo replay mode
  - languages other than JS/TS
- **v2 or later:**
  - difficulty scaling (which is why US-14 and US-25 serve the intro student instead)
  - remediation videos
  - persistent profiles
  - a mastery dashboard
  - caching
  - configurable interruption modes
  - other coding agents

---

## Traceability: requirement to stories

| Requirement | Stories |
|---|---|
| FR-0.1 | US-11 |
| FR-0.2 | US-06, US-11 |
| FR-0.3 | US-09, US-24, US-V2 |
| FR-0.4 | US-11, US-27 |
| FR-0.5 | US-08 |
| FR-0.6 | US-32 |
| FR-1.1 | US-08 |
| FR-1.2 | US-29, US-31 |
| FR-1.3 | US-03 |
| FR-1.4 | US-04 |
| FR-1.5 | US-06 |
| FR-1.6 | US-07, US-18 |
| FR-1.7 | US-04, US-05, US-07 |
| FR-2.1 | US-12, US-13 |
| FR-2.2 to FR-2.6 | US-13 |
| FR-2.3 | US-10, US-13 |
| FR-2.7 Explanation | US-14 |
| FR-2.7 Multiple choice | US-18 |
| FR-2.7 Written response | US-19 |
| FR-2.7 Quick-time event | US-20 |
| FR-2.7 System design graph, Data graphic | US-15 |
| FR-2.7 Video | US-V2 |
| FR-2.7 Prediction | US-21, US-29 |
| FR-2.7 Session summary | US-31 |
| FR-3.1 | US-11 |
| FR-3.2 | US-12 |
| FR-3.3 | US-14, US-16 |
| FR-3.4 | US-16 |
| FR-3.5 | US-20, US-22 |
| FR-3.6 | US-23 |
| FR-4.1 | US-18, US-20 |
| FR-4.2 | US-19, US-25 |
| FR-4.3 | US-25 |
| FR-4.4 | US-26 |
| FR-4.5 | US-24 |
| FR-5.1 | US-12, US-21 |
| FR-5.2, FR-5.3 | US-29 |
| FR-5.4 | US-21, US-29 |
| FR-6.1 | US-22, US-30 |
| FR-6.2 | US-30 |
| FR-6.3, FR-6.5 | US-31 |
| FR-6.4 | US-08 |
| FR-7.1, FR-7.2 | US-V1 |
| FR-7.3, FR-7.4 | US-V2 |
| FR-7.5, FR-7.6 | US-V3 |
| FR-8.1 | US-01 |
| FR-8.2, FR-8.3 | US-02 |
| FR-8.4 | US-02, US-V1 |
| NFR-1.1 | US-27 |
| NFR-1.2 | US-24, US-27 |
| NFR-1.3 | US-27, US-V3 |
| NFR-1.4 | US-33 |
| NFR-2.1 to NFR-2.3 | US-17 |
| NFR-2.2 | US-17, US-24 |
| NFR-3.1 | US-28 |
| NFR-3.2 | US-07, US-28, US-V3 |
| NFR-4.1, NFR-4.2 | US-32 |
| NFR-4.3 | US-09, US-32 |
| NFR-5.1, NFR-5.2 | US-10 |
| NFR-5.3 | US-04, US-06 |
| NFR-5.4, NFR-5.5, NFR-5.7 | US-09 (NFR-5.7 also US-V2) |
| NFR-5.6 | US-06, US-13 |
| NFR-5.8 | US-01 |
| NFR-6.1 | US-01, US-V1 |
| NFR-6.2 | US-03 |
| NFR-7.1 | US-13; every story through the definition of done |
| NFR-7.2 | US-33 |
| NFR-8.1 to NFR-8.3 | Every story through the definition of done; PBT targets in US-04, US-05, US-06, US-09, US-13, US-27, US-30 |
| AC-1, AC-2 | J1 |
| AC-3 | J1, US-18 to US-21, definition of done |
| AC-4 | US-24 |
| AC-5 | J2, US-V1 |
| AC-6 | J2, US-02, US-05 |

Every FR, NFR and AC in requirements.md maps to at least one story or journey.

## Story count and sizes

| Epic | Stories | S | M | L |
|---|---|---|---|---|
| E1 Set up Pi | 5 | 3 | 2 | 0 |
| E2 Pair with Claude Code | 5 | 3 | 2 | 0 |
| E3 Learn while Claude builds | 7 | 3 | 3 | 1 |
| E4 Answer exercises | 7 | 4 | 3 | 0 |
| E5 Recover from wrong answers | 2 | 1 | 1 | 0 |
| E6 Keep costs and failures in check | 2 | 2 | 0 | 0 |
| E7 End the session | 3 | 1 | 2 | 0 |
| E8 Behind the scenes | 2 | 2 | 0 | 0 |
| **Core total** | **33** | **19** | **13** | **1** |
| E9 Videos (Optional feature) | 3 | 1 | 1 | 1 |
