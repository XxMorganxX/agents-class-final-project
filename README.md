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

Pi draws on the current task, the code Claude changed, and what the developer has already demonstrated. In the messaging app, Pi might ask what happens when one person opens the app in two browser tabs. The answer depends on how the code associates a user with an open connection. A language model can interpret a developer’s explanation in their own words and identify the assumption behind a wrong prediction. Pi can then explain the relevant lines, ask a follow-up, or skip a concept the developer already understands. Pi will use Claude Haiku 5.5 through the Anthropic API to generate questions and give feedback. Pi can ask Claude Code to verify an explanation against the implementation.

## Scope for December 16

By December 16 we plan to run one Pi session alongside one Claude Code session on a laptop, with the two agents communicating in both directions. Claude tells Pi what it changed and why, and Pi asks Claude to explain or verify things it shouldn’t reason about on its own. While Claude builds and runs the code, Pi acts as a dynamic interface that explains important decisions and tests the student’s understanding in six ways:
- multiple-choice questions
- short-answer and teach-back exercises
- quick-time events such as predicting output or spotting a bug
- pseudocode and system design exercises
- generated explanation videos
- drill-down follow-up questions when an answer is wrong

Pi keeps a record of the concepts covered during the session. It stays cheap by using Claude Haiku 5.5 and simple logic, and leaves hard technical reasoning to Claude Code.

This scope is broad, but it builds on pieces we already have. A teammate has a working video generation engine, so Pi only needs to request videos asynchronously rather than build video generation. Every interaction type shares one pipeline: Pi detects a code change, picks a concept, generates an exercise, and evaluates the answer. Each new type is mostly a new prompt and a UI card on that shared pipeline. We’ll demo on a small app with behavior we can verify by running it. To assess whether Pi helped, we’ll ask the student to predict what the same code does in a new situation and compare their reasoning with the observed result.

Outside this release are difficulty scaling, videos tailored to a specific wrong answer, and detailed models of a student’s prior education. Learning state lasts one session. The [founding idea](docs/founding-idea.md) (§10) lists these along with ideas for later versions.

## Collaboration plan

We propose meeting twice a week on Zoom and using GitHub Issues between meetings to assign work, raise questions, and record decisions. We’ll devote the first meeting to the problem, intended users, and the scope we can deliver by December 16. During Inception, one teammate will operate Claude Code for the shared workflow while all five discuss its questions and review each document before approval. For Construction, we’ll divide the work into session integration, question generation, answer evaluation and learning records, the user interface, and testing the demo. Each task will have an owner and a second teammate to review it.

See the [founding idea](docs/founding-idea.md) and the [course requirements](docs/assignment-requirements.md). The [milestone map](docs/mid-semester-milestone.md) lists every milestone deliverable with its location and status.

## AI-DLC setup

- **Coding agent:** Claude Code
- **AI-DLC version:** v1.0.1 ([release](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1))
- **Rules:** `CLAUDE.md` is an unmodified copy of `core-workflow.md`, and the rule details are in `.aidlc-rule-details/`. Project context for the agent is in `.claude/CLAUDE.md`.
- **Artifacts:** AI-DLC writes everything to `aidlc-docs/`, including the `audit.md` and `aidlc-state.md` record.

To start, open Claude Code in this repo and begin a prompt with "Using AI-DLC, …". The agent should show the AI-DLC welcome message and create `aidlc-docs/`.
