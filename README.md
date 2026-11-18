# Anti-Vibe Coding

| Team member | Cornell NetID |
| --- | --- |
| Alexander Joos | aaj46 |
| Armand Larche | adl239 |
| Charlotte Bark | cpb222 |
| Morgan Stuart | mns66 |
| Zaid Patankar | zmp9 |

## Project overview

AI coding agents let developers build software they may struggle to explain or debug on their own. A developer might ask an agent to build a messaging app and see messages arrive instantly, yet have little idea how the app keeps track of who is connected. We’re building Anti-Vibe Coding for developers who want to keep the speed of working with a coding agent while gaining the judgment to evaluate its decisions. Claude Code builds and tests the software. Pi, a second agent, follows the session and helps the developer understand the engineering behind it.

Pi draws on the current task, the code Claude changed, and what the developer has already demonstrated. In the messaging app, Pi might ask what happens when one person opens the app in two browser tabs. The answer depends on how the code associates a user with an open connection. A language model can interpret a developer’s explanation in their own words and identify the assumption behind a wrong prediction. Pi can then explain the relevant lines, ask a follow-up, or skip a concept the developer already understands. For the prototype, we propose using Anthropic’s Claude Haiku 4.5 through the Anthropic API for questions and feedback. Pi can ask Claude Code to verify an explanation against the implementation.

## Scope for December 16

Our proposed demo follows the first phase of the founding idea. We plan to run one Pi session alongside one Claude Code session on a laptop. After a meaningful code change, Pi will generate a multiple-choice or short-answer question, evaluate the response, explain the answer, and keep a record of the concepts covered. We’ll use a small app with behavior we can verify by running it. To assess whether the explanation helped, we’ll ask the developer to predict what the same code does in a new situation and compare their reasoning with the observed result. Generated videos, timed challenges, and learning histories across projects fall outside this proposed first release.

## Collaboration plan

We propose meeting twice a week on Zoom and using GitHub Issues between meetings to assign work, raise questions, and record decisions. We’ll devote the first meeting to the problem, intended users, and the scope we can deliver by December 16. During Inception, one teammate will operate Claude Code for the shared workflow while all five discuss its questions and review each document before approval. For Construction, we’ll divide the work into session integration, question generation, answer evaluation and learning records, the user interface, and testing the demo. Each task will have an owner and a second teammate to review it.

See the [founding idea](docs/founding-idea.md) and the [course requirements](docs/assignment-requirements.md).

## AI-DLC setup

- **Coding agent:** Claude Code
- **AI-DLC version:** v1.0.1 ([release](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1))
- **Rules:** `CLAUDE.md` is an unmodified copy of `core-workflow.md`, and the rule details are in `.aidlc-rule-details/`. Project context for the agent is in `.claude/CLAUDE.md`.
- **Artifacts:** AI-DLC writes everything to `aidlc-docs/`, including the `audit.md` and `aidlc-state.md` record.

To start, open Claude Code in this repo and begin a prompt with "Using AI-DLC, …". The agent should show the AI-DLC welcome message and create `aidlc-docs/`.
