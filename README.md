# Anti-Vibe Coding

| Team member | Cornell NetID |
| --- | --- |
| Alexander Joos | aaj46 |
| Armand Larche | adl239 |
| Charlotte Bark | cpb222 |
| Morgan Stuart | mns66 |
| Zaid Patankar | zmp9 |

## Project overview

AI coding agents let developers build software they may struggle to explain or debug on their own. A developer might ask an agent to build a messaging app and see messages arrive instantly, yet have little idea how the app keeps track of who is connected. We’re building Anti-Vibe Coding for students learning to code with AI coding agents. These students are still building the fundamentals an agent lets them skip, so they are the most likely to ship code they can’t explain. We want them to keep the speed of working with a coding agent while gaining the judgment to evaluate its decisions. Claude Code builds and tests the software. Pi, a second agent, follows the session and helps the developer understand the engineering behind it.

Pi draws on the current task, the code Claude changed, and what the developer has already demonstrated. In the messaging app, Pi might ask what happens when one person opens the app in two browser tabs. The answer depends on how the code associates a user with an open connection. A language model can interpret a developer’s explanation in their own words and identify the assumption behind a wrong prediction. Pi can then explain the relevant lines, ask a follow-up, or skip a concept the developer already understands. Pi will use Claude Haiku 5.5 through the Anthropic API to generate questions and give feedback. Pi can ask Claude Code to check its answer keys against the implementation.

## Scope for December 16

By December 16 we plan to run Pi, a teaching assistant agent, alongside one Claude Code session on a laptop. Claude Code's hooks tell Pi what Claude changed and why. Pi asks Claude, read-only, to check the answers to its own questions before the student relies on them. Pi never changes the student's code. While Claude builds and runs the code, Pi appears in a desktop chat window (an Electron app built for learning, with the pi agent harness underneath).

Everything Pi shows in that window is a lesson component. The agent picks a type from a fixed set and fills it in, and it can't produce any other UI. The MVP has these types:
- explanations of the code Claude just wrote
- multiple-choice questions
- quick-time events, which are multiple-choice questions with a countdown
- written teach-back answers, graded against a checklist drawn from the actual code
- system design graphs and small data graphics, for display only
- prediction cards for the before-and-after check
- generated explanation videos
- a session summary

When an answer is wrong, Pi offers a short drill-down: up to two follow-ups and a retest.

The before-and-after check is the core of the product. Before Pi explains a concept, the student predicts what the code will do. When the session ends, they predict again in a new situation. The summary compares the two results, along with teach-back results, how much Pi interrupted and what the session cost. Pi keeps this record for one session only. It stays cheap by using Claude Haiku 5.5 and simple rules, and leaves hard technical reasoning to Claude Code.

We will build this in two weeks, which is feasible because every way of teaching shares one pipeline. Pi detects a code change, picks a concept, fills in a component and grades the answer, so each new component type is a schema and a renderer on that pipeline. Videos come from a teammate's working video engine (Project Feynman), which Pi runs in a background Claude session. They are off by default, and Pi turns them on only if the engine and its requirements are on the student's machine. Pi works fully without them. We'll demo live on a small WebSocket chat app whose behavior we can check by running it.

Outside this release are student-drawn diagrams and pseudocode editors, difficulty scaling, videos tailored to a specific wrong answer, and detailed models of a student's prior education. The [founding idea](docs/founding-idea.md) (§10) lists these along with ideas for later versions. [`requirements.md`](aidlc-docs/inception/requirements/requirements.md) (§3) records where the plan moved on from the founding idea.

## Collaboration plan

We propose meeting twice a week on Zoom and using GitHub Issues between meetings to assign work, raise questions, and record decisions. We’ll devote the first meeting to the problem, intended users, and the scope we can deliver by December 16. During Inception, one teammate will operate Claude Code for the shared workflow while all five discuss its questions and review each document before approval. For Construction, we’ll divide the work into session integration, question generation, answer evaluation and learning records, the user interface, and testing the demo. Each task will have an owner and a second teammate to review it.

See the [founding idea](docs/founding-idea.md) and the [course requirements](docs/assignment-requirements.md). The [milestone map](docs/mid-semester-milestone.md) lists every milestone deliverable with its location and status.

## AI-DLC setup

- **Coding agent:** Claude Code
- **AI-DLC version:** v1.0.1 ([release](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1))
- **Rules:** `CLAUDE.md` is an unmodified copy of `core-workflow.md`, and the rule details are in `.aidlc-rule-details/`. Project context for the agent is in `.claude/CLAUDE.md`.
- **Artifacts:** AI-DLC writes everything to `aidlc-docs/`, including the `audit.md` and `aidlc-state.md` record.

To start, open Claude Code in this repo and begin a prompt with "Using AI-DLC, …". The agent should show the AI-DLC welcome message and create `aidlc-docs/`.
