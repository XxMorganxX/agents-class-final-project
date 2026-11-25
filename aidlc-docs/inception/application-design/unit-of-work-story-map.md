# Unit of Work Story Map

**Stage:** INCEPTION - Units Generation (Part 2: Generation)
**Sources:** stories.md (36 stories with acceptance criteria), components.md (story coverage), `unit-of-work.md`, `plans/unit-of-work-plan.md` (Q2, Q9, Q12, CQ5, R4, R6, the decision rule)

## Mapping rule

- Each story maps to **exactly one** unit: the unit that holds the behavior most of its acceptance criteria check (Q2 A).
- Other units the story needs are listed as **supporting units**.
- When a criterion can only be checked after a later unit exists, it is checked **in that later unit's loop** (team decision rule). The last column says where. A story counts as done when its own unit is approved **and** every later check listed for it has passed.
- Criteria marked `[review]` in stories.md are hand reviews. They are done in the story's own unit unless the column says otherwise.

## Rule for a dropped U7 (team change, 2026-11-25)

U7 is optional and is built only if every core unit is approved by 2026-12-10 (Q10). Some criteria of core stories can only be checked in U7's loop. **If U7 is dropped, those video-only criteria are recorded as not applicable, and the stories close without them.** The decision to drop U7, and each criterion marked not applicable, is recorded in `audit.md`. If U7 is built, they are checked in U7's loop as listed.

The rule applies to these criteria, which are all the places a core story depends on U7:

| Story | Unit | Video-only criterion (not applicable if U7 is dropped) | Checked without U7 |
|---|---|---|---|
| US-07 | U2 | The teaching loop keeps working when the **video runner** fails | The Builder bridge part, in U2 |
| US-09 | U3 | `offer_video` is in the agent's tool list **when videos are on**; `make-video` is spawned with an argument array | The tool list with videos off, in U3; the other spawns in U3 and U2 |
| US-27 | U1b | The video cap is **enforced when a video is reserved**; video work uses the student's Claude plan, not Pi's API key | The 3-video cap value and its accounting, in U1b |
| US-28 | U3 | A **video runner** failure never stops the teaching loop | The Builder bridge part, in U3 |

Checked and **not** dependent on U7: US-27's "Videos: 3 per session" value (config and ledger), US-31's video count in the efficiency metric (a pure function over a ledger snapshot, tested in U5), and US-02's "the setup screen doesn't check video requirements" (a check U6 makes without U7).

**AC-5 and journey J2 step 5** ("turning videos on shows what is missing") also need U7 code (`checkRequirements` in C21). The team applied the same rule to them (Clarification Question 6 A): if U7 is dropped, that part is recorded as not applicable, with dated notes in requirements.md (AC-5) and stories.md (J2). The first half of AC-5, the full demo flow on a clean machine without the video runner, is always required.

**The video toggle (reading R7):** if U7 is dropped, Pi has no video setting at all (team answer to CQ6). So U6 shows the video toggle in settings only when the video plug-in is in the build. This follows components.md, where the toggle is "shown only to start C21's checks". It is checked in U6's loop: with no plug-in present, settings show no video toggle.

## Story map

| Story | Title | Size | Unit | Supporting units | Criteria checked in a later loop |
|---|---|---|---|---|---|
| US-01 | Add my API key | S | **U2** | U1a, U6 | U6: the missing-key message in the window; the key never reaches the renderer or preload API |
| US-02 | Get through first-run setup | M | **U6** | U2 | none |
| US-03 | Choose the repo Pi watches | S | **U2** | U1b, U6 | U6: the chosen path shows in the window |
| US-04 | Install Pi's hooks with my consent | M | **U2** | U1a, U6 | U6: the hooks step shows the exact entries |
| US-05 | Remove Pi's hooks cleanly | S | **U2** | U6 | U6: the Remove Pi hooks button |
| US-06 | Pi follows what Claude Code does | M | **U2** | U1b, U5 | none (C8 in U1b and C9 in U5 are built earlier, CQ5) |
| US-07 | Pi never slows or blocks Claude Code | S | **U2** | U3, U5, U7 | U7 (not applicable if U7 is dropped): the loop survives a video runner failure (the Builder part is checked in U2) |
| US-08 | My session lasts until I close Pi | S | **U5** | U3 | none |
| US-09 | Pi can't touch my repo or my secrets | M | **U3** | U2, U7 | U2: the argument-array spawn for `claude auth status`; U7 (not applicable if U7 is dropped): `offer_video` in the tool list with videos on, and the `make-video` spawn; Build and Test: `git status` shows no change after a full demo session |
| US-10 | The Pi window shows only lessons | S | **U6** | U4 | none |
| US-11 | As Pi, decide whether something is worth teaching | M | **U3** | U1a, U1b, U5 | U5: "a student response or End session" as triggers (the hook trigger is checked in U3) |
| US-12 | As Pi, choose the next lesson for a concept | M | **U3** | U5, U1b | U5: "before prediction first" and "explain first" use `nextLessonHint`, which U5 builds (M6) |
| US-13 | See every lesson as a card in one feed | L | **U1b** | U3, U5, U6, U4 | U3: regenerate once, then drop; U6: the chat-style feed with no free-text box; U6: adding a test-only type needs no change to the agent loop, feed, grading flow or record (the last of these exists only in U6) |
| US-14 | Get short explanations while Claude edits | M | **U4** | U3, U5 | U3: the snippet with file path and lines comes from the pipeline's snapshot, and the "file changed" note from `markCodeChanged`; U3: the two `[review]` criteria (terms defined, correct and pitched right) need real agent output; U5: the card shows during the turn (cadence) |
| US-15 | See diagrams and small charts | S | **U4** | U3 | U3: `[review]` the diagram matches the real code (needs real agent output) |
| US-16 | Get cards at a pace that doesn't break my flow | S | **U5** | U6 | U6: the in-window badge with no OS notification; a normal movable window |
| US-17 | Know Pi is working on a card | S | **U3** | U5, U6 | U5: the preparing indicator shows and clears; U6: the indicator appears in the window |
| US-18 | Answer a multiple-choice question | S | **U4** | U3, U5 | U3: `[review]` the answer key and distractors (AC-4 review); U5: only the first response is graded and saved to the record |
| US-19 | Explain it back in my own words | M | **U4** | U3, U5 | U3: the checklist is drafted in the same call that writes the question; U3: `[review]` the verdicts against a teammate's grades (live grader check, CQ4); U5: "partly" follow-up and "not yet" drill-down; the answer is kept when grading fails |
| US-20 | Answer a quick-time event | S | **U4** | U1a, U5 | U5: a timed-out question is recorded as skipped |
| US-21 | Predict what the code will do before Pi explains | M | **U4** | U1b, U5 | U5: the prediction is saved as "before", the actual result is added from a test run Claude already did (C8), and "no result" shows in the summary |
| US-22 | Skip an exercise without penalty | S | **U5** | U3, U4 | none |
| US-23 | Answer a question about code that changed | S | **U4** | U3, U6 | U3: the "code changed" note comes from `markCodeChanged`; U6: the note shows in the window |
| US-24 | As Pi, check answer keys with the Builder before asking | M | **U3** | U1b | none (AC-4 review here, two-day limit on prompt fixes; reruns in Build and Test if below 90%) |
| US-25 | Choose to see the answer or drill down | M | **U5** | U3, U4 | none |
| US-26 | Dispute a grade | S | **U5** | U4 | none |
| US-27 | Stay within the session's cost limits | S | **U1b** | U1a, U3, U5, U7 | U3: Builder calls and Haiku spend are charged through the ledger; U5: at the Haiku cap Pi stops, shows the notice once, and End session and the summary still work; U7 (not applicable if U7 is dropped): the video cap enforced when a video is reserved, and "video work uses the student's Claude plan" (the 3-video value and its accounting are checked in U1b) |
| US-28 | Keep going when a model call fails | S | **U3** | U5, U6, U7 | U5: the notice shows after retries fail and clears on the next success; U6: the notice appears in the window; U7 (not applicable if U7 is dropped): a video runner failure doesn't stop the loop |
| US-29 | Make my "after" predictions | M | **U5** | U3, U4, U6 | U6: the student enters the actual result in the card's field in the window |
| US-30 | Have my learning recorded fairly | S | **U5** | none | none |
| US-31 | See my session summary | M | **U5** | U4, U1b, U6 | U6: the summary is the only view of the record (no live record panel) |
| US-32 | Log what Pi did, without logging what the student wrote | S | **U1b** | U3, U6 | U3: every agent run logs its trigger, tool calls, outcome, timings and tokens; U6: a log write failure is reported once in the window |
| US-33 | Keep every prompt and limit in one config module | S | **U1a** | none | none (the "no prompt outside config" test runs in every later unit's Code Generation) |
| US-V1 | Turn on videos only when my machine can make them | M | **U7** | U6, U3 | none (U6 is built earlier) |
| US-V2 | Watch a short video about a concept when I want one | L | **U7** | U3, U5, U4 | none |
| US-V3 | Know my video limit and still learn if a video fails | S | **U7** | U1b, U6 | none |

## Stories per unit

| Order | Unit | Stories | Count | S / M / L |
|---|---|---|---|---|
| 1 | U1a Project Skeleton and Contracts | US-33 | 1 | 1 / 0 / 0 |
| 2 | U1b Shared Core | US-13, US-27, US-32 | 3 | 2 / 0 / 1 |
| 3 | U4 Lesson Component Types | US-14, US-15, US-18, US-19, US-20, US-21, US-23 | 7 | 4 / 3 / 0 |
| 4 | U3 Tutor Agent and Card Pipeline | US-09, US-11, US-12, US-17, US-24, US-28 | 6 | 2 / 4 / 0 |
| 5 | U5 Session and Learning Record | US-08, US-16, US-22, US-25, US-26, US-29, US-30, US-31 | 8 | 5 / 3 / 0 |
| 6 | U2 Builder Pairing and Setup | US-01, US-03, US-04, US-05, US-06, US-07 | 6 | 4 / 2 / 0 |
| 7 | U6 Desktop Shell and UI | US-02, US-10 | 2 | 1 / 1 / 0 |
| 8 | U7 Video *(optional)* | US-V1, US-V2, US-V3 | 3 | 1 / 1 / 1 |
| | **Total** | | **36** | **20 / 14 / 2** |

The core has 33 stories (19 S, 13 M, 1 L) and video has 3, which matches stories.md "Story count and sizes".

## Changes from the plan's draft story list

The approved plan listed draft stories per unit, with the final map made here. Applying the Q2 rule to the acceptance criteria moved two stories:

| Story | Draft unit | Final unit | Why |
|---|---|---|---|
| US-13 | U6 | **U1b** | Four of its six criteria check the registry: registration with schemas, only registered types, schema round-trips, and extensibility through the registry. One checks the pipeline (U3), and one checks the feed (U6) |
| US-27 | U3 | **U1b** | Its criteria check the cap values and cap accounting, which is C16 Budget Ledger, now in U1b. The cap-reached behavior is checked in U5's loop and the video cap in U7's |

U6 now owns two stories, but it closes the window-side checks of twelve earlier stories (see the table). U1b now carries three stories (one L). The work doesn't grow, because C3 and C16 were already in U1b. Most of the L in US-13 is cross-unit checks that run in later loops.

## Not stories: Build and Test

These aren't user stories, so they aren't mapped to a unit (R4). The testing and demo role owns them (Q9):

| Item | Checked in | Depends on |
|---|---|---|
| J1 The MVP demo session (AC-1, AC-2, AC-3) | Build and Test, then the live demo | every core unit |
| J2 Setup on a fresh clone without the video runner (AC-5, AC-6) | Build and Test. Step 5's video check only if U7 is built (CQ6 A) | U2, U6; U7 for step 5 |
| AC-1 Demo app built live with Claude Code | Live demo (not code written ahead of time) | every core unit |
| AC-2 Live demo only | Live demo | every core unit |
| AC-3 End to end on the demo app | Build and Test (Playwright end-to-end test) | every core unit |
| AC-4 Answer-key quality | U3's loop; reruns in Build and Test if U3 finished below 90% | U1b, U4, U3 |
| AC-5 Works without videos | Build and Test. The second half ("turning videos on shows what is missing") only if U7 is built; otherwise not applicable (CQ6 A) | every core unit; U7 for the second half |
| AC-6 Clean install | Build and Test | U2, U6 |

## Validation

- **Every story is mapped:** US-01 to US-33 and US-V1 to US-V3, 36 rows above.
- **Exactly one unit each:** each row names one unit, and the per-unit lists add up to 36 with no story twice.
- **No backward dependency for a story's own unit:** each story's supporting units that are built later appear only in the "later loop" column, never as something the story's own unit needs in order to build.
- **Optional stories are separable:** US-V1 to US-V3 are all in U7. The video-only criteria of US-07, US-09, US-27 and US-28 are checked in U7's loop. If U7 is dropped, they are recorded as not applicable and the stories close without them ("Rule for a dropped U7"). No other core story depends on U7. AC-5's second half and J2 step 5 follow the same rule (Clarification Question 6 A).
