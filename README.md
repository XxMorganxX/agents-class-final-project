# Anti-Vibe Coding

An interactive learning sidecar for AI coding agents. See
[`docs/founding-idea.md`](docs/founding-idea.md) for the concept and
[`docs/assignment-requirements.md`](docs/assignment-requirements.md) for the
course requirements.

> TODO for the milestone: team names and NetIDs, the 4–6 overview paragraphs
> (problem and LLM rationale, LLM integration plan, scope and feasibility), and
> the collaboration plan.

## AI-DLC setup

- **Coding agent:** Claude Code
- **AI-DLC version:** v1.0.1 ([release](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1))
- **Rules:** `CLAUDE.md` is an unmodified copy of `core-workflow.md`, and the rule details are in `.aidlc-rule-details/`. Project context for the agent is in `.claude/CLAUDE.md`.
- **Artifacts:** AI-DLC writes everything to `aidlc-docs/`, including the `audit.md` and `aidlc-state.md` record.

To start, open Claude Code in this repo and begin a prompt with
"Using AI-DLC, …". The agent should show the AI-DLC welcome message and
create `aidlc-docs/`.
