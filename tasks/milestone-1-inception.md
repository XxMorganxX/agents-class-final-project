# Milestone 1: Finish Inception and Submit

**Goal:** finish the AI-DLC Inception phase and submit the repo by **Tuesday, November 24, 2026, at 11:59 PM**.
**Rule:** do the steps in order. A step starts only when every "Done when" item of the step before it is true.
**Sources:** [`docs/mid-semester-milestone.md`](../docs/mid-semester-milestone.md), [`docs/assignment-requirements.md`](../docs/assignment-requirements.md)

## Already done
- [x] Setup, README, vision document and technical environment document
- [x] Workspace Detection
- [x] Requirements Analysis: approved 2026-11-23 and committed as checkpoint `b9b6ccb`

## Steps

### Step 1: Reset the agent's context before User Stories
- [x] Close this Claude Code session.
- [x] Open a new one in the repo and say: "Using AI-DLC, continue the workflow. Guide us through tasks/milestone-1-inception.md, one step at a time."

**Done when** the new session shows that it has resumed at User Stories from `aidlc-state.md`.

### Step 2: User Stories, part 1 (the plan)
- [x] The agent writes the story plan and its question file in `aidlc-docs/inception/plans/`. (The questions are embedded in `story-generation-plan.md`, as the AI-DLC user-stories rule requires.)
- [x] As a team, answer every `[Answer]:` tag in the question file. Leave none blank. (16 answers, plus 4 follow-ups all answered A, 2026-11-24)
- [x] Review the plan and approve it, or ask for changes. (approved 2026-11-24)

**Done when** every answer is filled in, the agent has checked the answers for ambiguity, and the team has approved the plan.

### Step 3: User Stories, part 2 (stories and personas)
- [x] The agent writes `user-stories/stories.md` (every story has acceptance criteria) and `user-stories/personas.md`.
- [x] Review both files as a team. Push back on anything that doesn't fit Pi. Graders look for engagement beyond approvals.
- [x] Approve. (2026-11-24)

**Done when** both files exist, the team has approved them, and `aidlc-state.md` marks User Stories complete.

### Step 4: Checkpoint and reset
- [x] Ask the agent to commit the checkpoint.
- [x] Reset the context, as in Step 1.

**Done when** the commit exists and a fresh session has resumed at Workflow Planning.

### Step 5: Workflow Planning
- [x] The agent writes `plans/execution-plan.md`.
- [x] Review which stages it runs or skips. Application Design and Units Generation must both run.
- [x] Approve. (2026-11-24)

**Done when** the execution plan is approved and `aidlc-state.md` marks Workflow Planning complete.

### Step 6: Checkpoint and reset
- [x] Commit the checkpoint, then reset the context. (committed and pushed; reset pending)

**Done when** the commit exists and a fresh session has resumed at Application Design.

### Step 7: Application Design
- [x] The agent writes the design plan and its question file. The team's more technical members lead these answers, since they decide how the code is organized and tested.
- [x] Answer every `[Answer]:` tag. (15 answers and 1 clarification)
- [x] Review the design files in `application-design/` and approve them. (approved after two team changes, 2026-11-25 UTC)

**Done when** the design files exist, the team has approved them, and `aidlc-state.md` marks Application Design complete.

### Step 8: Checkpoint and reset
- [x] Commit the checkpoint, then reset the context. (commit `5346244`, pushed)

**Done when** the commit exists and a fresh session has resumed at Units Generation.

### Step 9: Units Generation
- [x] Answer the units question file. (15 answers, 6 clarifications in three rounds: 5 before the plan was approved, the 6th during the units review; plan revised and approved 2026-11-25)
- [x] Review `application-design/unit-of-work.md`, `unit-of-work-dependency.md` and `unit-of-work-story-map.md`. Check that every story maps to a unit. (36 of 36 mapped; two changes requested and made: the dropped-U7 rule and the AC-5/J2 notes)
- [x] Approve the units and tell the agent: "Stop here. Do not start Construction." (2026-11-25)

**Done when** all three unit files exist, they are approved, `aidlc-state.md` marks Units Generation complete, and Construction has not started.

### Step 10: Final record check
- [x] The agent checks that `audit.md` logs every stage, answer and approval, and that `aidlc-state.md` matches the artifacts.
- [x] Every Inception artifact on the milestone list exists with real content.
- [x] The agent updates the status table in `docs/mid-semester-milestone.md`.

**Done when** nothing on the milestone checklist is missing or out of step with the record.

### Step 11: Commit and push
- [x] Commit everything that is left.
- [x] Confirm the push to GitHub. The agent asks first. (the team asked for the push, 2026-11-25)

**Done when** `git status` is clean and `main` on GitHub matches the local branch.

### Step 12: Submit
- [ ] Open the repo URL in a private or incognito window without signing in, and check that it loads.
- [ ] Submit the URL on Canvas.

**Done when** Canvas shows the submission.

## Progress log
- 2026-11-23: Created this list after Requirements Analysis was approved and committed. Starting Step 1.
- 2026-11-23: Step 1 done. A fresh session resumed at User Stories from `aidlc-state.md`. Step 2 started: wrote `user-stories-assessment.md` and `story-generation-plan.md` (16 questions). Waiting for the team's answers.
- 2026-11-24: New session resumed at User Stories Part 1. All 16 answers in `story-generation-plan.md` still blank. Step 2 is waiting on the team's answers.
- 2026-11-24: Step 2 done (16 answers, 4 follow-ups all A, plan approved). Step 3 done: stories.md (33 core + 3 video) and personas.md written and approved. Step 4: checkpoint committed; waiting for the context reset.
- 2026-11-24: Step 4 done: a fresh session resumed at Workflow Planning. Step 5: `execution-plan.md` written (Application Design and Units Generation both EXECUTE; Infrastructure Design SKIP). Waiting for the team's review and approval.
- 2026-11-24: Step 5 done: execution plan approved. Step 6: checkpoint committed and pushed; waiting for the context reset before Application Design.
- 2026-11-24 (2026-11-25 UTC): Step 7 done: Application Design plan answered (15 answers, 1 clarification), design files written, two team changes made, then approved. Step 8 done: checkpoint `5346244` committed and pushed, and a fresh session resumed at Units Generation. Step 9 started: wrote `plans/unit-of-work-plan.md` (draft of 7 units, 15 questions). Waiting for the team's answers.
- 2026-11-25 (UTC): Step 9 in progress. Plan answered (15 answers, 5 clarifications), revised before approval (U1 split into U1a and U1b, AC-4 time limit for U3) and approved. The three unit files are generated (8 units, 36 stories mapped). Waiting for the team to review the story map and approve the units.
- 2026-11-25 (UTC): Step 9 done. Units approved with R7 confirmed, and the team said "Stop here. Do not start Construction." aidlc-state.md marks Units Generation and Inception complete. Construction not started. Next: Step 10, the final record check.
- 2026-11-25 (UTC): Step 10 done. Record check found three gaps (an audit entry with no heading or timestamp, execution-plan.md progress out of date, no record of the bb0ff7a push); the team chose fixes and all three were applied as appended audit entries or logged changes. Milestone map rows 7 and 8 set to ✅. Step 11: everything committed and pushed. Next: Step 12, the incognito check and the Canvas submission.
