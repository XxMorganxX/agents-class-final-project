# Services

**Stage:** INCEPTION - Application Design
**Sources:** `components.md`, `component-methods.md`, `application-design-plan.md` (Q3, Q6, Q7, Q8, Q9 and the answer analysis), requirements.md, stories.md

Pi has no network services. "Service" here means a main-process component that coordinates other components for a user-visible flow. Everything runs in one Electron app on the student's laptop (NFR-6.2).

## Service list

| Service | Component | Coordinates | Owns |
|---|---|---|---|
| S1 Session Orchestrator | C9 | C11, C10, C15, C16, C18, C17 | Trigger intake, run queue, cadence, deterministic follow-ups, cap behavior, the feed, all writes to the learning record |
| S2 Card Pipeline | C10 | C3, C14, C13, C17 | Validation, regeneration, code snapshots, answer-key checks, latency limits |
| S3 Tutor Agent | C11 | C12, C13, C14, C10 | One agent run: prompt, tools, run limits |
| S4 Setup and Hooks | C7 with C6 | C6, C12, C22 | First-run checks, hook install and removal, runtime token file |
| S5 Hook Intake | C5 with C8 | C8, C9 | HTTP in, token and schema checks, repo filter, event normalization |
| S6 Video (optional) | C21 | C11 tool set, C3, C16, C9 | Requirement checks, video offers, render queue, fallback |

## S1 Session Orchestrator

The orchestrator is the center of a session (Q7 A). It is the only component that writes to the feed (through `FeedSink`) and to the learning record. Other components return results to it instead of acting on their own.

### Triggers and the run queue (Q8 A)
- There are three trigger sources (FR-0.2): Builder activity from S5, student responses from the window, and End session.
- At most one agent run is active at a time.
- While a run is busy, new Builder events wait in a pending list. When the run ends, they are merged with `mergeActivity` into one `builder-activity` trigger.
- Student responses and End session are never merged and keep their order (analysis 7). If both are waiting, they go before merged Builder activity, so a student never waits behind Claude's edits.
- `FileChanged` events also go straight to `CardPipeline.markCodeChanged` without waiting for the queue, so the "code changed" note appears at once (US-23).

### Cadence (FR-3.3)
- Before each run, the orchestrator passes the turn state to the agent: whether the turn has ended, and how many exercises this turn has shown.
- When the agent proposes a card, `mayShow` decides again. An exercise before `Stop`, or a third exercise in one turn, is dropped and logged (US-16).
- Explanations and display-only cards may show during the turn.
- The turn counter resets on the next `PromptSent` event.

### After a student response (Q6 B)
1. The window sends `respond(cardId, response)`. The response is checked against the type's response schema. Only the first response to a card is graded (US-18).
2. The grader runs:
   - Right/wrong grading is deterministic.
   - Checklist grading is one `generateJson` call. It counts toward the Haiku cap and is not an agent run (analysis 4).
3. The grade is appended to the record and pushed to the card.
4. `nextRecoveryStep` decides what comes next:
   - **Without the agent:** "Show me the answer", skip, dispute, adding an actual result, and drill-down step counting.
   - **With an agent run** (the student-response trigger): the "partly" follow-up, drill-down questions and the retest. These need new content.

### End session (Q3 C)
1. The student presses **End session**.
2. The orchestrator empties the queue, ignores further Builder activity, and works out the concepts covered from the record.
3. **If the Haiku cap isn't reached:** it starts an `end-session` agent run for each covered concept to show a new-situation "after" prediction card. These runs don't count toward the per-turn exercise limit.
4. **If the cap is reached:** it skips those runs. Concepts without an "after" card show "after: not asked (cap reached)" (analysis 3, US-27).
5. When every "after" card is answered or skipped, it calls `buildSummary` (a pure function) and shows a `session-summary` card. The summary uses no model call.

### Caps (NFR-1)
- Before each run, the orchestrator checks the ledger. If the Haiku cap is reached, it starts no run and shows the cap notice once.
- Builder and video caps are enforced where they are spent (C14, C21) and only disable those features.
- Drill-down at a cap: the next step becomes "reveal answer" (US-25).

### Status and failures (NFR-2.3, NFR-3)
- A "preparing" status shows from the moment a run starts until it ends.
- If `TutorModel` returns a failure after its own brief retries, the orchestrator shows the model notice. It clears the notice on the next successful call. No student action is needed (US-28).
- Builder and video failures come back as typed results. They never throw into the loop (NFR-3.2).
- Retry timing and backoff are left to NFR Design.

## S2 Card Pipeline

The pipeline runs inside the `show_component` tool call, then hands the result to S1.

1. **Concept check:** drop the card if its concept isn't on the config list (FR-3.1).
2. **Schema check:** `registry.validateFields`. On failure, call `regenerate` once with the errors. If that fails too, drop the card as `invalid-twice` and log it (FR-2.4).
3. **Snapshot:** for types that refer to code, read the line range and file hash through `RepoReader` and store them on the card (Q14 A).
4. **Answer-key check (Q9 C):**
   - If the card isn't flagged, go on to step 5.
   - If it carries a receipt that `receiptCovers` (verdict "agrees" and the hash matches the final question and key), go on without a second Builder call.
   - Otherwise call `BuilderBridge.checkAnswerKey`. "Agrees": go on. "Disagrees", timeout or cap reached: drop the card and log it. The student sees nothing broken (US-24).
5. **Latency limit:** if the time since the trigger is over 15 s (60 s for Builder-checked cards), drop the card and log it (US-17).
6. Return the `CardRecord` to S1, which applies cadence and shows it.

## S3 Tutor Agent

One run:
1. Build the request:
   - the system prompt from config
   - a trigger message for the trigger kind
   - `summarizeForAgent(events)`: labels, next-lesson hints, questions already asked or skipped
   - the turn state
2. Tools: `read`, `grep`, `find`, `ls` (path-guarded), `ask_builder`, `show_component`, and `offer_video` only when S6 is active.
3. Call `TutorModel.runAgent` through the budget decorator.
4. The run ends when:
   - `show_component` returns a shown card
   - the agent stops with no action ("nothing to teach", with its reason)
   - the step limit is reached
   - the run timeout passes
   - a cap is reached

   If the run ends any other way, nothing half-finished is shown (US-11).
5. Log the run to C17: trigger, tool calls, outcome, timings, tokens (FR-0.6).

`ask_builder` use (Q9 C): the agent may confirm a runtime-dependent key while writing. It then passes the returned `receiptId` in `show_component`. If it skips this, the pipeline checks the card in step 4.

## S4 Setup and Hooks

### Launch
1. The composition root validates config. An invalid value stops Pi and names the value (US-33).
2. It reads `.env`. A missing key shows the missing-key screen, and teaching doesn't start (US-01).
3. It writes the runtime file: port and a new random token, mode `0600` (clarification 1).
4. It starts the hook server.
5. It opens the window.
6. If setup has never passed, or the repo is gone, the window opens on the setup screen.

### Setup screen (FR-8.2)
The steps run in order: repo, hooks, Claude Code, API key. Each returns `StepResult`, which is pushed with `setup:changed`. A failed step names what is missing and lets the student retry.

### Hooks step (FR-1.7)
1. `preview()` shows the exact entries.
2. Nothing is written until the student confirms.
3. `install(true)` takes a byte-exact backup, then merges Pi's entries.
4. If the file is invalid JSON or the student cancels, nothing is written.

### Remove Pi hooks
`remove()` deletes only Pi's entries, or restores the backup when nothing else has changed (rule in FD). If no Pi entries are found, it says so.

### Quit
1. Stop the hook server.
2. Delete the runtime file.
3. Cancel any video job.
4. Flush the event log.

The hook entries stay in place. They do nothing while Pi is closed: the script finds no runtime file, or the post fails within its timeout, and it exits 0 (US-07).

## S5 Hook Intake

1. The hook script posts the payload with the token from the runtime file.
2. The server checks the token: `401` on a mismatch, logged.
3. It checks the schema: `400` on failure, logged.
4. It replies `202` at once.
5. Off the request path, `toBuilderEvent` drops events from outside the repo or from irrelevant tools. For `Stop`, `readTurnDetails` reads the end-of-turn message and test runs from the transcript Claude Code already wrote (FR-1.5). Pi runs nothing.
6. The event goes to `SessionOrchestrator.handleBuilderEvent`.

## S6 Video (optional)

1. The student turns videos on in settings.
2. `VideoPlugin.enable` runs `checkRequirements`. If anything is missing, it returns the missing items, and videos stay off (FR-7.2).
3. On success, it registers `offer_video`, the video component types and the `pi-video://` protocol.
4. When the agent calls `offer_video`, a `video-offer` card appears. Nothing renders until the student accepts (FR-7.3).
5. When the student accepts, the plug-in reserves a video in the ledger. At the cap, it offers nothing more (US-V3).
6. It enqueues a job and shows a `video-rendering` card.
7. When the job finishes:
   - **Success:** the rendering card is replaced by a `video` card.
   - **Failure:** the rendering card is replaced by an `explanation` card for the same concept, written by an agent run (FR-7.6).

   The session keeps going while a video renders.
8. Turning videos off calls `disable()`, which unregisters everything the plug-in added.

## Orchestration patterns used

| Pattern | Where | Why |
|---|---|---|
| Single coordinator (mediator) | S1 | One owner of cadence, caps, the feed and the record (Q7 A) |
| Serial work queue with coalescing | S1 run queue | Bursts of edits become one run (Q8 A); no two runs write at once |
| Pipeline | S2 | Ordered checks, each able to drop a card with a logged reason |
| Decorator | `BudgetedTutorModel` | No model call can skip the Haiku cap (analysis 9) |
| Event sourcing | C15 | Labels, pairs and metrics are pure functions over events (Q13 A) |
| Plug-in | S6 | The optional feature adds tools and types without core changes (Q15 A, FR-2.6) |
| Result types instead of exceptions | C12, C14, C21 | Failures in the Builder or video runner never break the loop (NFR-3.2) |
