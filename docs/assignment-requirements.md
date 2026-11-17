# Assignment Requirements: Class Project

Source: Canvas, "Class Project: Mid-semester Milestone"
(https://canvas.cornell.edu/courses/91792/assignments/1036177). This is a
structured summary written for this repo, not a verbatim copy. **The Canvas
page is authoritative.** If this file and Canvas disagree, Canvas wins.

## The project in one line

As a team, use an AI coding agent guided by **AI-DLC v1.0.1** to design and
build a product that itself uses LLMs. Agentic designs, where the LLM picks
steps and calls tools, are encouraged. Aim for a focused MVP. The product only
needs to run locally for a demo.

## Two graded stages

| Stage | Due | Weight | What's done |
|---|---|---|---|
| Mid-semester Milestone | Tue Nov 24, 2026, 11:59 PM | 15% (150 pts) | Repo + AI-DLC setup, README, vision doc, tech-env doc, full **Inception** phase |
| Final Project | Wed Dec 16, 2026 | 35% | **Construction** for every unit of work, demo video, recorded presentation, repo, peer evaluation |

AI-DLC's Operations phase is not required.

## Hard constraints

- **AI-DLC version must be v1.0.1**, not the v2 that is now on AI-DLC's `main` branch.
  - Release: https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1 (`ai-dlc-rules-v1.0.1.zip`)
  - Setup per agent: https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/README.md
  - Claude Code layout: core rules go in `CLAUDE.md` (a copy of `core-workflow.md`), and rule details go in `.aidlc-rule-details/`.
  - Commit the rule files.
  - Smoke test: a prompt beginning "Using AI-DLC, …" should show the AI-DLC welcome message and create `aidlc-docs/`.
- AI-DLC artifacts live in `aidlc-docs/`. Application code lives at the repo root, never inside `aidlc-docs/`.
- Never commit API keys or secrets.
- The choice of coding agent and of LLM provider is not graded. Name the planned model and provider in the README and the tech-env doc.
- Run **one shared AI-DLC workflow as a team**, not one per member.

## Milestone deliverables (the repo must contain)

1. AI-DLC v1.0.1 rule files for each coding agent the team uses.
2. `README.md` with:
   - every member's full name and NetID
   - the coding agent(s) used and the AI-DLC version (v1.0.1)
   - about 4–6 paragraphs (1–2 pages) covering:
     1. the problem, who it's for, and why an LLM fits
     2. the planned LLM integration: which features use it, agents or tools, model and provider
     3. scope and feasibility by the final deadline: meaningful, but achievable
     4. the collaboration plan: meeting cadence, in person or online, communication between meetings, and how work is split
3. **Vision document** covering: what is being built and for whom, MVP features in scope, explicitly out-of-scope features, and open questions.
4. **Technical environment document** covering:
   - language and version, package manager, framework
   - cloud provider and deployment model (state that it runs locally, and how)
   - test framework
   - a prohibited-libraries table, with a reason and an alternative per entry
   - security basics
   - at least one example each of a typical endpoint, a function, and a test
   - No placeholders. The whole team reviews and agrees on both docs before Inception starts.
5. Inception artifacts in `aidlc-docs/inception/`:
   - `requirements/requirements.md` and `requirements/requirement-verification-questions.md`, with every `[Answer]:` filled in
   - `user-stories/stories.md` and `user-stories/personas.md`
   - `plans/execution-plan.md`
   - `application-design/*`
   - `application-design/unit-of-work.md`, `unit-of-work-dependency.md`, `unit-of-work-story-map.md`
6. The AI-DLC record: `aidlc-docs/audit.md` and `aidlc-docs/aidlc-state.md`.

**Stop after Units Generation.** Approve the units, then tell the agent not to
enter Construction until the final project.

## Process rules

- Reset the agent's context at each gate.
- Prefix exploratory questions with "Do not update any documents." If the agent leaves one out of `audit.md`, ask it to add the entry.
- Fix problems through the workflow. Don't hand-edit AI-DLC artifacts or the record.
- If something is missing from `audit.md`, ask the agent to add it. Don't edit the file yourself.
- Answer every question deliberately, and push back or request changes when a proposal doesn't fit. Graders look for engagement beyond plain approvals.
- AI help on the README and input docs is allowed, but the team owns the content.

## Submission

Push everything to GitHub and submit the repo URL on Canvas. The link must
open in a private or incognito window without signing in, so the repo must be
public or otherwise viewable without login.

## Rubric (150 pts, five criteria at 30 each)

| Criterion | What earns full marks |
|---|---|
| A. LLM product proposal and scope | README makes a compelling case for LLMs, gives a well-defined integration plan, and sets a scope that is meaningful yet achievable |
| B. Input documents | Vision and tech-env docs are complete, specific and actionable, with useful example code |
| C. AI-DLC process and record | Correct v1.0.1 setup; every AI request answered; active engagement; every artifact produced through the workflow. **Incorrect setup caps this at Fair (≤22).** |
| D. Inception artifacts | Every artifact present and high quality; requirements consistent with the vision; stories with acceptance criteria; specific personas; plan, design and units fit together |
| E. Clarity | Names and NetIDs, a collaboration plan, and an organized repo. All or nothing, except 10 pts if only the collaboration plan is missing. |
