# Personas: Anti-Vibe Coding (Pi, the learning sidecar)

**Stage:** INCEPTION - User Stories (Part 2: Generation)
**Sources:** requirements.md §4 (users), founding-idea.md, story-generation-plan.md (Q1 B, Q2 C, Q3 C, follow-up 1 A)

Three personas and two anti-personas. The personas are invented. They use they/them pronouns. Requirements §4 sets the limits that hold for all of them:
- Pi pitches its questions at the intermediate level and defines every technical term the first time it uses it.
- The student's code is JavaScript or TypeScript.
- Pi runs on macOS, locally, beside Claude Code.

The demo app in every scenario is the WebSocket chat server from AC-1.

---

## P1: Riley, the intro-level student

| Field | Profile |
|---|---|
| Background | Second-semester CS student, done with CS1 in JavaScript. Has built a to-do app and a small Express API by following tutorials. This is their first project built mostly with Claude Code |
| Goal | Finish the course project without having to say "I don't know what this does" when the instructor asks |
| Goal | Learn the words people use (event, callback, socket) well enough to read Claude's end-of-turn messages |
| Goal | Know which parts of the code they actually understand |
| Frustration | Claude writes 200 lines in a minute, and it all looks correct, so they accept it |
| Frustration | Tutorials explain things in the abstract, not in the code on their screen |
| Frustration | Asking a basic question feels embarrassing, so they don't ask |
| Uses Claude Code today | Types a goal, accepts most edits and runs what Claude says to run. Rarely opens the files Claude changed |
| Knows | Variables, functions, arrays and objects, `if` and loops, `npm install`, what a server is |
| Doesn't know yet | Event emitters and `.on()` handlers, closures over connection state, `async`/`await` versus callbacks, what a WebSocket handshake is, `Map` versus object, why a `Set` of clients exists |
| Reacts to interruptions | Welcomes them as long as each card is short. Leaves a card open about a minute before skipping. Loses heart after two wrong answers in a row with nothing to help |
| A good session | They can explain in their own words how a message gets from one client to all the others, and the summary shows they got better at predicting what the code does |

**How in-scope features serve Riley** (follow-up 1 A; Pi doesn't change difficulty per student):
- Pi defines every term the first time it uses it.
- After a miss, the next lesson on that concept explains first (FR-3.2).
- Drill-down gives two focused follow-ups and a retest (FR-4.3).
- A skip costs nothing (FR-3.5).

If those aren't enough for Riley, difficulty scaling (v2) is the fix. This release adds nothing for that.

**Day in the life.** Riley opens Pi beside their terminal and picks the `chat-server` folder. They ask Claude Code to "add a WebSocket server that broadcasts messages to everyone." While Claude edits `server.ts`, an explanation card appears in Pi: "`wss.on('connection', …)` registers a *handler*, a function the server calls each time a new client connects." Before any explanation of broadcasting, a prediction card asks what happens when client A sends "hi" while B and C are connected. Riley guesses "only B gets it," which is wrong. Claude's turn ends, and a multiple-choice card about the `clients` set appears. Riley misses it and picks **Drill down**. Pi asks two smaller questions and gives a retest with a new example, and Riley gets the retest right. Riley skips a quick-time event they don't feel ready for. At the end, Riley presses **End session** and answers the "after" prediction correctly. The summary shows *broadcast* moving from wrong to right, *closures* still "shaky", and three skipped cards.

---

## P2: Devin, the intermediate student

| Field | Profile |
|---|---|
| Background | Junior CS major. Built two full-stack projects in TypeScript, one with React. Uses Claude Code daily for coursework and a part-time web job |
| Goal | Use Claude Code at full speed without losing track of the design choices it makes |
| Goal | Be able to defend the architecture in a code review or interview |
| Goal | Spend little time on things they already know |
| Frustration | Notices weeks later that Claude chose a pattern they wouldn't have, and nobody asked them |
| Frustration | Explanations that start from zero waste their time |
| Frustration | Tools that interrupt while they are in the middle of steering Claude |
| Uses Claude Code today | Writes detailed prompts, reads diffs, runs tests and pushes back on Claude's choices |
| Knows | `async`/`await`, closures, event emitters, TypeScript types, Express, basic testing |
| Doesn't know yet | Backpressure and buffered sends, heartbeat and ping/pong for dead connections, the trade-offs of room-based fan-out, handling `close` versus `error` |
| Reacts to interruptions | Tolerates cards only at the end of Claude's turn. Skips explanations of things they know within seconds. Values "which approach would you have picked?" questions |
| A good session | Few cards, each about a real decision in the code. Right on the "after" predictions. A summary that says the session cost little |

**Day in the life.** Devin asks Claude Code to add rooms and a heartbeat to the chat server. Pi stays quiet while Claude edits boilerplate. Pi shows one short explanation when Claude adds `ws.ping()`. At the end of the turn, Pi shows a multiple-choice card: "Claude stores rooms as `Map<string, Set<WebSocket>>`. Which would you have picked?" It then compares Devin's pick with what Claude built. A written-response card asks Devin to explain why a dead connection would never be removed without the heartbeat. Pi grades it "partly": Devin hit the timeout point but missed that `close` never fires. One follow-up closes the gap. Devin disputes one multiple-choice grade they think is unfair. The summary shows two cards per turn, nothing skipped, and $0.04 of Haiku spent.

---

## P3: Sam, the demo teammate

| Field | Profile |
|---|---|
| Background | A member of the Pi team who runs the live demo (AC-2) and checks the clean install (AC-6). Comfortable in a terminal. Didn't write most of the setup code |
| Goal | Clone the public repo, follow the README and reach a working session without asking a teammate |
| Goal | Show every response component type in one live session (AC-3) |
| Goal | Leave their own `~/.claude/settings.json` exactly as it was afterwards |
| Frustration | Setup steps that fail with no message |
| Frustration | A demo machine without a Metal GPU or the video runner |
| Frustration | Not knowing whether a quiet Pi is broken or just waiting |
| Uses Claude Code today | Daily, with their own hooks and settings already in place |
| Knows | The Pi requirements, the demo script and the concept list |
| Doesn't know yet | How the setup code handles their own existing hooks |
| Reacts to interruptions | During the demo, wants cards to appear quickly and predictably (NFR-2) so the video flows |
| A good session | Setup is green on the first try. Every response type appears. Removing the hooks restores their settings file exactly |

**Day in the life.** Sam clones the repo on a laptop without Project Feynman. They copy `.env.example` to `.env` and paste the API key. They launch Pi, and the setup screen shows four steps. Sam picks the demo repo and reads the exact hook entries Pi will add, then confirms. The Claude Code check and the API key check pass. Sam turns videos on to show what happens, and Pi names the missing `VIDEO_ENGINE_DIR` and leaves videos off. Sam records the live demo session. Afterwards they press **Remove Pi hooks** and diff `~/.claude/settings.json` against their backup. Nothing differs.

---

## Anti-personas

Pi isn't built for these users. They explain the "won't" criteria in the stories.

### A1: Casey, who wants Pi to do the work

A student who wants Pi to write or fix the code, or to give answers to paste into an assignment. Pi doesn't serve Casey:
- Pi's tools only read (FR-0.3).
- Pi never sends anything to Claude Code's live session.
- The student can't type free questions (FR-2.5).

Stories that show this: US-09, US-10, US-13.

### A2: Morgan, the experienced developer

A senior engineer who already understands the code and sees every card as a cost. Pi doesn't change its teaching for Morgan. Morgan explains why these exist:
- the two-exercise limit per turn and the end-of-turn cadence (FR-3.3)
- free skips (FR-3.5)
- no OS notifications or stolen focus (FR-3.4)
- Pi never blocking Claude (FR-1.6)
- the interruption-cost metric (FR-6.5)

Stories that show this: US-07, US-16, US-22, US-31.

---

## Persona-to-story map

Filled in after the stories were written. ● = the story's main persona, ○ = also served.

| Story | P1 Riley | P2 Devin | P3 Sam | Pi (agent) | Pi developer |
|---|---|---|---|---|---|
| US-01 API key | ○ | ○ | ● | | |
| US-02 Setup screen | ○ | ○ | ● | | |
| US-03 Choose repo | ● | ○ | ○ | | |
| US-04 Install hooks | ○ | ○ | ● | | |
| US-05 Remove hooks | | ○ | ● | | |
| US-06 Follow Claude Code | ● | ○ | ○ | | |
| US-07 Never block Claude | ○ | ● | ○ | | |
| US-08 Session lifetime | ● | ○ | | | |
| US-09 Repo stays untouched | ○ | ● | ○ | | |
| US-10 Safe window | ○ | ● | | | |
| US-11 Decide whether to teach | | | | ● | |
| US-12 Choose the next lesson | | | | ● | |
| US-13 Lesson feed | ● | ○ | ○ | | |
| US-14 Live explanations | ● | ○ | | | |
| US-15 Diagrams and graphics | ● | ○ | | | |
| US-16 Pacing | ○ | ● | | | |
| US-17 Preparing indicator | ○ | ○ | ● | | |
| US-18 Multiple choice | ● | ● | | | |
| US-19 Written teach-back | ○ | ● | | | |
| US-20 Quick-time event | ● | ○ | ○ | | |
| US-21 Before prediction | ● | ○ | | | |
| US-22 Skip | ● | ● | | | |
| US-23 Code changed | ○ | ● | | | |
| US-24 Builder answer-key check | | | | ● | |
| US-25 Show answer or drill down | ● | ○ | | | |
| US-26 Dispute a grade | ○ | ● | | | |
| US-27 Session caps | ○ | ● | ○ | | |
| US-28 Model failures | ○ | ○ | ● | | |
| US-29 After prediction | ● | ○ | | | |
| US-30 Learning record | ● | ○ | | | |
| US-31 Session summary | ● | ● | ○ | | |
| US-32 Logging | | | | | ● |
| US-33 Config module | | | | | ● |
| US-V1 Turn on videos | ○ | | ● | | |
| US-V2 Offer and render video | ● | | ○ | | |
| US-V3 Video limit and fallback | ● | | ○ | | |

Every persona maps to at least one story, and every story to at least one persona. A1 and A2 have no stories by design. They are listed above with the stories that show their limits.
