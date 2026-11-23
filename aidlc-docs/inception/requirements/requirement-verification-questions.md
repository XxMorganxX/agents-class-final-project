# Requirements Verification Questions

**Stage:** INCEPTION - Requirements Analysis
**Project:** Anti-Vibe Coding (Pi, the learning sidecar)
**Inputs read:** `docs/founding-idea.md` (vision), `docs/technical-environment.md`, `README.md`, `docs/assignment-requirements.md`

## How to answer

Answer each question by writing the letter of your choice after its `[Answer]:` tag. If no option fits, choose the last option (Other) and describe what you want after the tag. You can add a short note after any letter, for example `[Answer]: B, but cap it at two follow-ups`.

Please answer every question. The answers decide what goes into `requirements.md`, and later the user stories, design and units of work.

## What these questions leave alone

The vision and technical environment documents already decide the following, so no question reopens them:

- Pi is an Electron desktop app beside Claude Code, built in TypeScript on Node.js 24 with the pi agent SDK, Fastify, TypeBox and React.
- Pi uses Claude Haiku 5.5 and escalates hard reasoning to the Builder through read-only `claude -p`.
- Pi runs locally only, and is read-only toward the student's code.
- Learning state lasts one session. Difficulty scaling, remediation videos and long-term profiles are out of scope.
- Videos are optional, off by default, run through the student's own Claude login, and are capped at 3 per session.

Each question below covers something those documents leave silent, vague, contradictory, or list as an open question. The "Why we ask" line points to the section that leaves it open.

## Team directions given outside the numbered questions

These came from the team during the question round. They are requirements, and they shape several answers below.

1. **Two-week build window (2026-11-20).** "This is a 2-week-long project." Construction must fit in two weeks, so the MVP scope is sized to that.
2. **Lesson component abstraction (2026-11-20).** Everything Pi shows the student, or asks of them, is a typed *lesson component*: for example a multiple-choice question, a written-response question, a generated video, a system design graph, an on-the-fly data graphic or a quick-time event. Pi's agent can only pick from a fixed set of component types and fill in their fields. It never sends free-form UI. The Pi window renders the components in order as a chat-style feed. This is a narrow agent-to-UI system: adding a new way to teach means adding a component type, not changing the agent loop or the window.
3. **Videos are real but optional (2026-11-20).** The video component ships in the MVP and plays real videos made by Project Feynman (make-video). Pi makes them by running a Claude session in the background, but only when the video runner is installed on the student's machine. The repo is public, so Pi must work fully without the runner. Pi checks for the runner's requirements. If the student turns videos on and something is missing, Pi says what is missing and leaves the feature off.

---

# Part 1: Users and context

## Question 1
Which student level should Pi pitch its explanations and questions at?

*Why we ask:* Founding idea §1 targets CS students "still building the fundamentals" but names no year or experience level. §10 rules out difficulty scaling, so the MVP needs one fixed level, and the personas must match it.

A) Intro students (CS1 to CS2): comfortable with basic syntax, new to multi-file projects, async code and web servers

B) Intermediate students (sophomores and juniors, past data structures): building their first full-stack or systems projects with Claude Code

C) Mixed intro and intermediate: questions pitched at the intermediate level, with explanations that define every term on first use

D) Other (please describe after [Answer]: tag below)

[Answer]: C. Questions pitched at the intermediate level; explanations define every term on first use. Team accepted AI proposal (2026-11-20, "accept all").

## Question 2
Which programming languages in the student's repo must Pi handle well in the MVP?

*Why we ask:* The technical environment fixes Pi's own stack (TypeScript), but no document says what language the student's code is in. That choice shapes the prompts, the recorded hook fixtures (tech-env §5) and the code viewer. Question 32 (the demo app) should match this answer.

A) JavaScript/TypeScript only

B) Python only

C) Both JavaScript/TypeScript and Python, each with its own test fixtures

D) Any language on a best-effort basis, with only the demo app's language tested

E) Other (please describe after [Answer]: tag below)

[Answer]: A. JavaScript/TypeScript only. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 2: Functional scope and interaction types

## Question 3
If time runs short before December 16, what is must-have and what gets cut first?

*Why we ask:* Founding idea §10 and the README list live explanations, six interaction types and two-way Builder communication as in scope, with no priority among them. Units Generation needs that order to decide what to build first and what can slip.

A) Everything in §10 is must-have. The fallback order is to reduce polish first, then cut videos

B) Must-have: live explanations, multiple choice, short answer/teach-back, drill-down and the two-way Builder link. If needed, cut videos first, then quick-time events, then pseudocode/system design

C) Must-have: everything except videos. Videos are nice-to-have because they are optional, off by default and need a Mac with a Metal GPU

D) Must-have: live explanations, multiple choice and drill-down only. Every other type is nice-to-have

E) Keep all interaction types. Reduce Pi-to-Builder calls to answer verification only, and drop Builder explanation and concept-identification requests first

F) Other (please describe after [Answer]: tag below)

[Answer]: F) Other. Sized for the 2-week build. Must-have: the lesson component system; explanation, multiple-choice, written-response (teach-back), quick-time event (multiple choice with a countdown), system design graph (display only), data graphic and session summary components; the video component with real Project Feynman generation as an optional, requirement-gated feature; drill-down; the before-and-after check; Builder verification of answer keys. Stretch only: student-drawn diagrams, and Builder calls for explanations or concept identification. Team accepted AI proposal (2026-11-20, "accept all").

## Question 4
How do live explanations relate to quizzes and other exercises?

*Why we ask:* §3A (explanations) and §3B to §3F (exercises) describe each type on its own but not how they connect. The §7 walkthrough opens with a diagnostic question before any explanation, while §6's passive mode explains without asking anything.

A) Teach then test: Pi explains a decision first, then asks about that same concept

B) Test first: Pi asks first, and the explanation comes as feedback after the answer

C) Separate tracks: explanations stream on their own and need no reply. Exercises are chosen separately and may cover concepts Pi never explained

D) Pi decides per concept: a diagnostic question for a new concept, and an explanation first for a concept the student already missed this session

E) Other (please describe after [Answer]: tag below)

[Answer]: D. Diagnostic question first for a new concept; explanation first for a concept already missed this session. Team accepted AI proposal (2026-11-20, "accept all").

## Question 5
How does a drill-down on a wrong answer end?

*Why we ask:* Founding idea §4 and §10 say Pi asks focused follow-ups and then retests with a new example. They don't say how many rounds to run, what happens if the retest also fails, or whether the student can stop early.

A) Fixed length: up to two follow-ups and one retest. Pi then shows the full answer, marks the concept "still shaky" and moves on

B) Pi continues until the student passes a retest, with a "show me the answer" button to leave at any time

C) One simpler follow-up, then Pi reveals and explains the answer. The retest with a new example comes later in the same session

D) The student chooses after a wrong answer: "just show me the answer", or "drill down", which runs up to two follow-ups and one retest and then reveals the answer

E) Other (please describe after [Answer]: tag below)

[Answer]: D. The student chooses "just show me the answer" or "drill down" (up to two follow-ups and one retest, then reveal). Team accepted AI proposal (2026-11-20, "accept all").

## Question 6
Can the student start a conversation with Pi, for example by typing "why did Claude use a Map here?" into the Pi window?

*Why we ask:* Founding idea §6 and the README call Pi's window a "chat window", but §3 and §10 describe only Pi-initiated explanations and exercises. Whether the student can ask Pi questions changes the UI, the prompts, the cost cap and the user stories.

A) No. Pi only starts interactions. The student answers cards and cannot type free questions

B) Yes, but only about the card on screen (for example "I don't get option B"), answered by Haiku

C) Yes, any question about the current code. Haiku answers simple ones and passes harder ones to the Builder through read-only `claude -p`

D) Yes, any question, and every student question goes straight to the Builder. Haiku only formats the answer

E) Other (please describe after [Answer]: tag below)

[Answer]: A. Pi starts every interaction. The feed looks like a chat, but the student replies only through each component's own controls. Card-scoped student questions (option B) are a possible later component type. Team accepted AI proposal (2026-11-20, "accept all").

## Question 7
How does Pi decide which concepts are worth teaching or testing, and what counts as "the same concept" in the session record?

*Why we ask:* Founding idea §3A says to prioritize concepts that are "important, unfamiliar, or likely to affect future decisions", but §10 keeps no profile of prior knowledge, so Pi can't know what is unfamiliar. No document says whether concepts come from a fixed list or are named freely by the model, which decides whether the session record can merge repeat appearances of one concept.

A) A fixed list the team writes (for example 30 to 50 concepts such as "async/await ordering" or "race condition"). Haiku tags each change with concepts from that list only

B) Open-ended: Haiku names concepts freely from the code, and Pi merges near-duplicate names into one record

C) Ask the Builder: Pi uses a read-only `claude -p` call to name the concepts in each change that matter most, and Haiku only writes the exercises

D) A fixed list for the record and the demo, and Haiku may add new concepts that are marked "unlisted"

E) Other (please describe after [Answer]: tag below)

[Answer]: A. A fixed team-written list of about 20 concepts, chosen for the WebSocket demo app. Haiku tags changes from that list only. Team accepted AI proposal (2026-11-20, "accept all").

## Question 8
When the student has turned videos on, who decides when a video is made?

*Why we ask:* Founding idea §3D says videos are "generated selectively", and §10 says they explain concepts "as the code is built", but neither says who triggers them. Tech-env §4 puts each video at 2 to 3 minutes of render time on the student's Claude plan, with a cap of 3 per session, so the trigger rule decides how those 3 are used.

A) Pi decides on its own, choosing the concepts it judges hardest, until the session cap is reached

B) Pi offers a video on a card ("Want a 45-second video on this?") and renders it only if the student accepts

C) The student requests one from any explanation card with a "Make a video" button. Pi never starts one on its own

D) Pi offers, and the student can also request. Both count against the same cap

E) Other (please describe after [Answer]: tag below)

[Answer]: B. Pi offers a video on a card and renders it only if the student accepts. Applies only when the video feature is enabled and its requirements are met. Team accepted AI proposal (2026-11-20, "accept all").

## Question 9
How does the student answer a pseudocode or system design exercise in the MVP?

*Why we ask:* Founding idea §3F says the student "can write pseudocode, draw a component architecture, or explain a design decision", and §6 lists architecture diagrams and pseudocode editors as components. §10 keeps these exercises in scope without saying whether drawing is included, and a diagram editor is a large build compared with a text box.

A) Text only: a pseudocode editor with a monospace font and line numbers, plus a plain explanation field

B) Text plus a simple structured form: the student lists components and the connections between them, and Pi renders that as a read-only diagram

C) Text plus a drag-and-drop diagram canvas where the student draws boxes and arrows

D) Other (please describe after [Answer]: tag below)

[Answer]: D) Other. Not applicable in the MVP: there is no dedicated pseudocode or system-design answer input. Diagrams are display-only. A written-response component may ask for pseudocode as plain text. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 3: Pi and Builder integration

## Question 10
Should Pi ever make Claude Code wait for the student, and how do "before Claude builds it" exercises such as "choose the architecture" work?

*Why we ask:* Founding idea §13.2 and tech-env §9 leave blocking the Builder undecided, and tech-env §9 notes that hook timeout behavior is not yet confirmed. §3E ("before Claude implements a new component") and §3F ("before or after Claude implements") assume exercises that come before Claude acts, which only work cleanly if Claude can be held. For options C to E: if testing shows a hook cannot hold Claude reliably, Pi falls back to never blocking.

A) Never block. Every exercise is about code Claude already wrote, and "choose the architecture" asks which approach the student would have picked and compares it with Claude's

B) Never block, but when the student sends a prompt, Pi may pose a design question while Claude keeps working, then compare the answer with what Claude built

C) Block only before major new features: Pi holds Claude through a hook until the student answers a design question or presses Skip

D) Gate the next prompt: if an exercise is still open when the student sends Claude a new prompt, that prompt waits until they answer or skip

E) Blocking is a switch in the Pi window, off by default. When it is on, Pi gates the next prompt as in option D

F) Other (please describe after [Answer]: tag below)

[Answer]: A. Never block. "Choose the architecture" compares the student's pick with what Claude built. A blocking switch (option E) is out of MVP scope. Team accepted AI proposal (2026-11-20, "accept all").

## Question 11
Founding idea §10 says Claude sends Pi its "key decisions" and test results. Which trade-off is acceptable for getting them?

*Why we ask:* The hooks in tech-env §1 and §8 cover file edits, turn end and prompt submit, so nothing yet supplies decisions or test results. Each way of getting them has a cost: weaker detail, use of the student's Claude plan, or a change to the read-only rule. The exact mechanism is left to Application Design; this question sets the constraint.

A) Use only what Claude Code already produces (Claude's end-of-turn message and any test commands it ran). No extra cost and no change to the Builder, but decisions are only as clear as Claude's wording

B) Pi may spend one read-only Builder call per Claude turn to get a short summary of decisions and test outcomes. Richer, but it uses the student's Claude plan every turn

C) Setup may add one instruction to the student's project so Claude ends each turn with a short structured note for Pi. Cheap and precise, but it changes how the Builder behaves and is an exception to "Pi never writes to the student's repo" (tech-env §7)

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Use only what Claude Code already produces: the end-of-turn message, the transcript and the test commands it ran. Team accepted AI proposal (2026-11-20, "accept all").

## Question 12
Which generated questions must the Builder (read-only `claude -p`) verify before the student relies on them, and what does the student see meanwhile?

*Why we ask:* Founding idea §9 says complex questions need a verified answer, §10 puts answer verification in scope, and tech-env §8 has Haiku set `needsBuilderVerification`. Neither says which questions qualify, what the student sees during a call that can take tens of seconds, or what happens on disagreement or when the Builder-call cap (Question 25) is reached.

A) Only questions Haiku flags. Pi holds each one until the Builder confirms it, and drops it if the Builder disagrees, times out or the cap is reached

B) Only questions Haiku flags, shown right away and marked "unverified". If the Builder later disagrees, Pi corrects the answer key and tells the student

C) Every question whose answer depends on how the code runs (predict the output, spot the bug, concurrency) is held until verified, whatever Haiku's flag says

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Hold Haiku-flagged questions until the Builder confirms them; drop on disagreement, timeout or cap. Team accepted AI proposal (2026-11-20, "accept all").

## Question 13
May Pi add its hook settings to the student's repo when it is first set up?

*Why we ask:* Tech-env §2 says hook entries are "installed into the student's repo", but §7 says "Pi never writes to the student's repo". No document resolves this. The exact install steps are left to Application Design; this question sets the rule.

A) Yes, as the one exception to the read-only rule: Pi writes the hook entries to a gitignored local settings file in the student's repo, after showing them to the student and getting a click to confirm

B) No repo files change: the hooks go into the student's user-level Claude Code settings once, and Pi ignores events from any folder other than the repo chosen in the Pi window

C) No automated install: the README gives the hook snippet and the student pastes it in by hand

D) Other (please describe after [Answer]: tag below)

[Answer]: B. Team direction (2026-11-19): "Pi should not write code to the repo, it is meant to explain the code and ensure understanding of the project with the human." Pi changes no files in the student's repo.

## Question 14
What counts as one session, both for pairing Pi with Claude Code and for how long learning state lasts?

*Why we ask:* Founding idea §10 pairs one Pi session with one Claude Code session and keeps learning state for one session, but never defines a session. Hook events carry Claude's `session_id` (tech-env §8), which changes when Claude Code restarts or the student runs `/clear` while the Pi window stays open.

A) One launch of the Pi app. State lasts until Pi is closed, even if Claude Code restarts or `/clear` is used

B) One Claude Code session. A new `session_id` in the hook events ends the old session and starts fresh learning state

C) The student starts and ends a session with buttons in the Pi window, and events outside an active session are ignored

D) Other (please describe after [Answer]: tag below)

[Answer]: D) Other. One launch of the Pi app (survives Claude Code restarts and /clear), plus an "End session" button that runs the "after" half of the before-and-after check and shows the session summary. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 4: User experience and interruption

## Question 15
Configurable interruption modes are deferred to v2, so the MVP ships one fixed cadence. When does Pi show explanations and exercises, and how often?

*Why we ask:* Founding idea §8.2 says to trigger after "meaningful changes" without defining them, §10 defers the §6 passive/interactive/deep modes to v2, and tech-env §1 posts an event after every edit. Nothing caps how many exercises a session gets beyond §8.7's "avoid unnecessary assessments".

A) Once per Claude turn: Pi waits for Claude to finish, looks at all the edits together, and shows at most one explanation and one exercise

B) Mixed: short explanations appear live as significant files are edited, and exercises wait for the end of Claude's turn, at most one or two per turn

C) Live: Pi may show an explanation or exercise after any single edit it judges significant, with at least a few minutes between exercises

D) Explanations appear live, and exercises go into a queue with a badge that the student opens when ready

E) The mixed cadence in option B, plus a longer checkpoint assessment when a feature looks finished

F) Other (please describe after [Answer]: tag below)

[Answer]: B. Mixed: short explanations live as significant files are edited; exercises at the end of Claude's turn, at most one or two per turn. Team accepted AI proposal (2026-11-20, "accept all").

## Question 16
Where does the Pi window sit on screen?

*Why we ask:* Founding idea §13.1 and tech-env §9 both list window placement as still open, naming a side-docked window and always-on-top as options.

A) A normal window the student places anywhere, beside the terminal

B) Docked to the right edge of the screen at a fixed width

C) A small always-on-top panel that expands when a card arrives and collapses after it is answered

D) Other (please describe after [Answer]: tag below)

[Answer]: A. A normal window the student places anywhere. Team accepted AI proposal (2026-11-20, "accept all").

## Question 17
How does Pi tell the student that a new card needs them?

*Why we ask:* Founding idea §13.1 and tech-env §9 ask how Pi "gets the student's attention without breaking their flow", and list system notifications as one option. Founding idea §6 calls this "the central design challenge".

A) Only inside the Pi window, with a highlight or badge

B) A macOS notification when a card needs an answer

C) The Pi window brings itself to the front

D) An in-window badge for explanations, and a macOS notification only for exercises that need an answer

E) Other (please describe after [Answer]: tag below)

[Answer]: A. In-window badge or highlight only. Team accepted AI proposal (2026-11-20, "accept all").

## Question 18
What can the student do when they don't want to answer a question right now?

*Why we ask:* Founding idea §6 calls protecting the student's flow "the central design challenge" and §11 measures interruption cost, but no section says whether questions can be skipped, deferred or snoozed, or how a skip is recorded.

A) Nothing: each question must be answered before the next one appears

B) Skip only. A skip is recorded as "skipped", not wrong, and that question is not asked again this session

C) Skip or "ask me later". Deferred questions come back at the end of Claude's next turn

D) Skip and "ask me later", plus a snooze button that stops all new cards for a set time

E) Other (please describe after [Answer]: tag below)

[Answer]: B. Skip only, recorded as "skipped", not wrong, and not asked again this session. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 5: Learning state and grading

## Question 19
Within one session, how should Pi label how well the student understands each concept?

*Why we ask:* Founding idea §4 describes five levels plus mastery and evidence, and §9 says mastery must not be inferred from one multiple-choice answer, but §10 only says Pi "records which concepts came up and how the developer answered". Requirements must say whether a label exists and how it is earned.

A) No labels. The record lists each concept with the questions asked and how the student did, with no summary level

B) The five levels from §4 (encountered, recognize, explain, apply, repeatedly misunderstands), set by fixed rules from which exercise types the student passed

C) A simpler three-level label (not yet shown, shaky, solid) set by fixed rules, where "solid" needs correct answers in at least two different exercise types

D) Haiku picks the label after each answer and stores a one-line reason as evidence

E) Other (please describe after [Answer]: tag below)

[Answer]: C. Three levels (not yet shown, shaky, solid) set by fixed rules; "solid" needs correct answers in at least two different exercise types. Team accepted AI proposal (2026-11-20, "accept all").

## Question 20
How are answers graded, and what result does the student see?

*Why we ask:* Founding idea §3C says to grade understanding rather than wording, §3F asks for feedback on correctness, assumptions and edge cases, and §9 lists five evaluation dimensions. None defines a pass, partial credit, or which result starts the §10 drill-down.

A) Right or wrong for every exercise type. Any answer that is not right starts a drill-down

B) Multiple choice and quick-time events are right or wrong. Free-text answers (teach-back, pseudocode, system design) get a three-level verdict (got it / partly / not yet) with short feedback. "Partly" gets one targeted follow-up and "not yet" starts a full drill-down

C) Same as option B, but before asking a free-text question Pi drafts a checklist of key points from the actual code. The verdict comes from that checklist, and the student sees which points they hit or missed

D) A score on each dimension (correctness, reasoning, assumptions, edge cases) plus written feedback, for every free-text answer

E) Other (please describe after [Answer]: tag below)

[Answer]: C. Checklist-based grading for free-text answers, with the checklist drafted from the actual code in the same model call that writes the question. Team accepted AI proposal (2026-11-20, "accept all").

## Question 21
What happens when Pi's grade of a student's free-text answer might be wrong?

*Why we ask:* Founding idea §5 has Pi grade only "simple responses" and escalate hard reasoning, and §9 lists confidence as something to evaluate. No document says what Pi does when Haiku is unsure of a grade, or how a student can challenge one. (Whether the answer key itself is correct is covered by Question 12.)

A) Show Haiku's verdict. The student can mark it "disputed" in the session record, without a recheck

B) Haiku reports its confidence, and below a set threshold Pi asks the Builder to grade the answer before showing a result

C) A "Pi got this wrong" button on every graded card. The Builder rechecks the grade, and if Pi was wrong the attempt is voided and a correction is shown

D) Both the confidence check in option B and the button in option C

E) Other (please describe after [Answer]: tag below)

[Answer]: A. Show Haiku's verdict; the student can mark it "disputed" in the session record, without a recheck. Team accepted AI proposal (2026-11-20, "accept all").

## Question 22
If the code a question is about changes while the question is on screen, what should Pi do?

*Why we ask:* Founding idea §9 says that when the implementation changes, "existing questions and explanations may need to be invalidated", and Claude keeps editing while cards are open. No document defines when a question is stale or what the student sees.

A) Freeze it: the question and snippet stay as asked, grading uses the code from that moment, and the card notes that the code has since changed

B) Withdraw it: Pi marks the card outdated, does not grade it, and writes a new question on the new code if the concept still applies

C) Recheck it: Haiku decides whether the change affects the correct answer. Pi keeps the question if not and withdraws it if so

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Freeze the question and note that the code has since changed. Team accepted AI proposal (2026-11-20, "accept all").

## Question 23
What does the student see of the session's learning record, during and after the session?

*Why we ask:* Founding idea §10 keeps session-scoped concept records in scope but moves the concept-mastery dashboard to v2, while §6 lists progress visualizations as a UI component. No document says what the MVP shows or what the student keeps when the session ends.

A) Nothing during the session. A summary card at the end lists the concepts covered, how each went and which are still weak

B) A live panel listing each concept seen with a simple status (seen, answered right, needs work), plus the end-of-session summary card

C) Only a small counter in the window header, for example "6 concepts, 4 correct"

D) The live panel and summary card, plus a Markdown report saved in `.pi-sidecar/` that the student can keep. Pi never reads it in later sessions

E) Other (please describe after [Answer]: tag below)

[Answer]: A. No live panel. A session summary component at the end of the feed lists concepts, results and the before-and-after outcome. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 6: Non-functional requirements

## Question 24
What spending cap should Pi enforce on its own Haiku usage per session? (Builder calls and videos use the student's Claude plan and are covered in Question 25 and the video cap.)

*Why we ask:* Tech-env §7 says `config.ts` caps Haiku tokens per session but gives no value. Founding idea §8 says to maximize learning per dollar but names no budget, and the budget limits how many exercises Pi can generate. The estimates below use Claude Haiku 5.5's list price ($0.10 per million input tokens, $0.50 per million output tokens) and assume roughly $0.005 to $0.01 per exercise once generation, grading and Pi's own file reads are counted. requirements.md will restate them once the team picks one.

A) $0.10 per session (about 10 to 20 exercises). Pi stops intervening when it is reached

B) $0.25 per session (about 25 to 50 exercises). Pi stops intervening when it is reached

C) $1.00 per session (about 100 to 200 exercises, more than a session needs). Pi stops intervening when it is reached

D) No dollar cap. Keep only a high token ceiling as a safety stop, and show the running spend in the window

E) Other (please describe after [Answer]: tag below)

[Answer]: B. $0.25 per session. Team accepted AI proposal (2026-11-20, "accept all").

## Question 25
How many read-only `claude -p` Builder calls may Pi make per session, given that each one uses the student's Claude plan and can take tens of seconds?

*Why we ask:* Tech-env §7 says `config.ts` caps Builder calls per session but gives no value. Founding idea §5 and §10 depend on these calls for verification and hard explanations, so the cap shapes verification, drill-down and the student's plan usage.

A) Low: up to 5 per session, used only for answer verification

B) Medium: up to 15 per session, shared by verification, explanations and concept identification

C) High: up to 40 per session, with the remaining count shown in the Pi window

D) No fixed default. The student sets the cap in Pi's settings, starting at 15

E) Other (please describe after [Answer]: tag below)

[Answer]: B. Up to 15 Builder calls per session. In the MVP these are used for answer-key verification; video generation has its own cap of 3 per session. Team accepted AI proposal (2026-11-20, "accept all").

## Question 26
After Claude finishes a turn or makes a meaningful change, how quickly must Pi's first explanation or exercise appear?

*Why we ask:* Founding idea §8 asks for minimal added latency, and §3E quick-time events only work if they are timely, but no document sets a number. The target decides whether Pi prepares exercises during Claude's turn, and gives acceptance tests a threshold.

A) Within 5 seconds

B) Within 15 seconds

C) Within 30 seconds

D) No hard target. Show a "Pi is preparing an exercise" indicator and display the card when it is ready

E) Other (please describe after [Answer]: tag below)

[Answer]: E) Other. Within 15 seconds for explanations and unverified exercises; within 60 seconds for Builder-verified exercises; a "preparing" indicator shows while waiting. Team accepted AI proposal (2026-11-20, "accept all").

## Question 27
When a model call fails (Haiku errors, times out or is rate-limited, or a `claude -p` call fails), how should Pi behave toward the student?

*Why we ask:* Tech-env §7 covers only invalid input (logged and dropped) and spending caps (Pi stops intervening). No document covers model failures, which decides whether a flaky network during the demo looks like a silent Pi or a visible error. Note: if you opt in to the Resiliency Baseline (Question 29), its rules may add stricter requirements on top of this answer.

A) Retry briefly in the background, then silently skip that intervention

B) Retry briefly, then show a small status notice such as "Pi can't reach the model, retrying", and resume automatically when calls work again

C) Fall back to content that needs no model, such as the changed code with its file and function names and a reflection prompt

D) Pause Pi and show an error card until the student clicks Retry

E) Other (please describe after [Answer]: tag below)

[Answer]: B. Retry briefly, show a small status notice, resume automatically. Team accepted AI proposal (2026-11-20, "accept all").

## Question 28
What should Pi's local log record, and how long should it stay on the laptop?

*Why we ask:* Tech-env §6 names a JSONL event log and §7 keeps answers in `.pi-sidecar/`, while founding idea §10 limits learning state to one session. Neither says what the log contains or when it is cleared. Debugging grading needs full prompts, but the privacy of the student's code and answers argues for keeping less.

A) Metadata only (event types, timings, token counts, errors), deleted when the session ends

B) Metadata plus full prompts, model outputs and student answers, kept in `.pi-sidecar/` until the student deletes it

C) Metadata always. Full prompts, outputs and answers only when a debug setting is on, kept until the student deletes it

D) Everything (metadata, prompts, outputs, answers), deleted automatically when the session ends

E) Other (please describe after [Answer]: tag below)

[Answer]: C. Metadata always; full prompts, outputs and answers only with a debug setting on. Stored in Pi's own app-data folder, never in the student's repo. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 7: AI-DLC extensions

These three questions come from the AI-DLC v1.0.1 extension files. Each extension, if enabled, adds rules that every later stage must follow.

## Question 29: Resiliency Extensions
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of **directional, design-time best practices** for building resilient systems, derived from the **AWS Well-Architected Framework (Reliability Pillar)** and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability — covering 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does **not** make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a **starting point** that scaffolds good resiliency decisions early — it is not a substitute for a formal **AWS Well-Architected Review** of the built system.

Treat the output as a well-grounded **first draft of your resiliency posture** to build on and validate — not a finished, production-certified result.

A) Yes — apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live)

B) No — skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability)

X) Other (please describe after [Answer]: tag below)

[Answer]: B. No. Local prototype. Team accepted AI proposal (2026-11-20, "accept all").

## Question 30: Security Extensions
Should security extension rules be enforced for this project?

*Context:* Tech-env §7 already sets security basics for Pi (Electron hardening, a localhost-only hook server with a session token, read-only tools, no shell strings, secrets kept out of the window). This question is whether the full AI-DLC security rule set is enforced on top of those, as blocking checks at every stage.

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)

B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)

X) Other (please describe after [Answer]: tag below)

[Answer]: B. No. The tech-env §7 security controls (Electron isolation, localhost hook token, read-only Builder tools, safe subprocess calls) are kept as explicit non-functional requirements instead. Team accepted AI proposal (2026-11-20, "accept all").

## Question 31: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)

B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)

C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)

X) Other (please describe after [Answer]: tag below)

[Answer]: B. Partial: PBT-02, PBT-03, PBT-07, PBT-08 and PBT-09 enforced; other PBT rules advisory. Targets: schema round-trips, component validation, mastery rules, cap accounting. Team accepted AI proposal (2026-11-20, "accept all").

---

# Part 8: Demo and success criteria

## Question 32
What small app will Claude Code build during the December 16 demo, so Pi's exercises have real code to work on?

*Why we ask:* The README's December 16 scope says only that the demo uses "a small app with behavior we can verify by running it". The founding idea uses a WebSocket chat (§7) and a cache (§3B) as examples but never picks one, and the choice drives the fixtures, prompts and demo script. Pick an app in a language you chose in Question 2.

A) A real-time WebSocket chat server in TypeScript, the §7 example (connection maps, two tabs, broadcast)

B) A small REST API with an in-memory cache in TypeScript, the §3B example (concurrent misses, invalidation)

C) A small Python command-line app, such as a task tracker that saves to a file, close to intro coursework

D) A small algorithm module with tests, such as a rate limiter or LRU cache, whose outputs are easy to predict and check

E) Other (please describe after [Answer]: tag below)

[Answer]: A. Real-time WebSocket chat server in TypeScript. Team accepted AI proposal (2026-11-20, "accept all").

## Question 33
How will the team show that Pi actually helped the student understand the code?

*Why we ask:* The README says the team will "ask the student to predict what the same code does in a new situation and compare their reasoning with the observed result", but founding idea §10 lists before-and-after comprehension checks under v2. The documents disagree on whether Pi or the team runs this check. Pi itself can't run code (it is read-only, and sandboxed execution is v3).

A) Built into Pi: at the end of a session Pi poses a new-situation prediction. The student runs the code, or the result comes from a test run the Builder already did, and Pi compares the prediction with it. This moves the v2 before-and-after check into the MVP

B) A study run by the team outside the product: a few classmates do the same task with and without Pi, answer the same transfer questions, and the team scores them by hand

C) Shown in the demo only: the video includes one before-and-after prediction, with no formal measurement in the MVP

D) Both the built-in check in option A and the small study in option B

E) Other (please describe after [Answer]: tag below)

[Answer]: A. Team direction (2026-11-19): "before-and-after is important and hsould be core." The before-and-after comprehension check moves from v2 into the MVP as a core feature.

## Question 34
What has to be true for the team to call the MVP done?

*Why we ask:* Founding idea §11 lists success metrics but sets no MVP targets, and tech-env §5 tests only schema shape with a fake model plus one end-to-end flow, with no requirement that answer keys are correct. requirements.md needs acceptance criteria to check against.

A) Working end to end: in one session on the demo app, every must-have interaction type appears at least once with no crashes, and the automated tests pass

B) Option A plus a hand-reviewed check: teammates review 20 to 30 exercises generated from recorded hook fixtures, and at least 90% have a correct answer key for the actual code

C) Option B plus measured targets: a full demo session stays under the Haiku cap chosen in Question 24, and each card appears within the latency target chosen in Question 26 (or within 30 seconds if no target was set)

D) Option A plus an automated check: in the opt-in live test suite, a stronger model (the Builder through `claude -p`) grades Haiku's exercises against the code

E) Other (please describe after [Answer]: tag below)

[Answer]: E) Other. Option B with 20 hand-reviewed exercises (at least 90% correct answer keys), plus the before-and-after check working end to end in one session. Team accepted AI proposal (2026-11-20, "accept all").

## Question 35
Will the demo video show a live Claude Code session, or Pi driven by recorded hook events?

*Why we ask:* Tech-env §5 records hook payloads as test fixtures, but neither document says whether the demo runs live. A replay mode would be a product requirement of its own, while a live run means accepting whatever Haiku and the Builder produce on the day.

A) Live: a teammate uses Claude Code on the demo app in real time, and the video shows Pi's real responses

B) Replay: recorded hook events drive Pi so the demo is repeatable, and Claude Code does not run during the recording

C) Live with a backup: record a live run, and also build a replay mode as a requirement in case the live output is poor

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Team direction (2026-11-20): "No replay demo. This is a 2-week-long project." The demo is a live run only. No replay mode is built for the demo; recorded hook fixtures stay for automated tests only (tech-env §5).
