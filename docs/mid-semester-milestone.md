# Mid-semester Milestone: Deliverables Map

Due **Tuesday, November 24, 2026, 11:59 PM**. We submit one link to this repository. This page lists every required item, where it lives in the repo, and its status. The full assignment text is in [`assignment-requirements.md`](assignment-requirements.md), and the Canvas page is authoritative.

Status key: ✅ done · 🟡 started, gaps remain · ⬜ not started

## 1. What we submit

| # | Deliverable | Location | Rubric | Status |
|---|---|---|---|---|
| 1 | AI-DLC v1.0.1 rule files | [`CLAUDE.md`](../CLAUDE.md), `.aidlc-rule-details/` | C (setup) | ✅ |
| 2 | README: names, NetIDs, agent, AI-DLC version | [`README.md`](../README.md) | E | ✅ |
| 3 | README: overview (problem and LLM rationale, LLM integration, scope) | [`README.md`](../README.md) | A | ✅ pending team review |
| 4 | README: collaboration plan | [`README.md`](../README.md) | E | ✅ |
| 5 | Vision document | [`founding-idea.md`](founding-idea.md) | B | ✅ pending team review |
| 6 | Technical environment document | [`technical-environment.md`](technical-environment.md) | B | 🟡 draft, pending team review |
| 7 | Inception artifacts | `aidlc-docs/inception/` | D | ⬜ |
| 8 | AI-DLC record | `aidlc-docs/audit.md`, `aidlc-docs/aidlc-state.md` | C | ⬜ |

## 2. Vision document: founding idea vs. requirements

The founding idea serves as our vision document. It covers the concept in depth, and now has all four required sections.

| Required section | In the founding idea? | Gap |
|---|---|---|
| What we're building and for whom | Yes (§1–3) | Done: target users are students learning to code with AI coding agents (§1). |
| MVP features in scope | Yes (§10) | Done: the in-scope list for Dec 16. |
| Features explicitly out of scope | Yes (§10) | Done: an out-of-scope list plus v2/v3 future versions. |
| Open questions | Yes (§13) | Done: web app vs. terminal, blocking Claude during Pi sessions, Pi–Claude communication, and the set of interaction types. |

The MVP is broad: six interaction types plus video. Criterion A grades whether the scope is achievable by Dec 16, so the README's scope paragraph should explain why it is feasible.

## 3. Technical environment document: required sections

- Language and version, package manager, framework
- Cloud provider and deployment model: runs locally (say how, e.g. a local web app that calls a model through Ollama or a hosted API)
- Test framework
- Prohibited-libraries table, with a reason and an alternative for each entry
- Security basics
- At least one example each of a typical endpoint, function, and test

The planned LLM model and provider also go in the README.

## 4. Inception artifacts AI-DLC must generate

All of these go under `aidlc-docs/inception/`:

- [ ] `requirements/requirements.md`
- [ ] `requirements/requirement-verification-questions.md`, with every `[Answer]:` filled in
- [ ] `user-stories/stories.md`, with acceptance criteria
- [ ] `user-stories/personas.md`
- [ ] `plans/execution-plan.md`
- [ ] `application-design/`: the design files
- [ ] `application-design/unit-of-work.md`
- [ ] `application-design/unit-of-work-dependency.md`
- [ ] `application-design/unit-of-work-story-map.md`

**Stop after Units Generation.** Approve the units and tell the agent not to start Construction.

## 5. Process rules (criterion C)

- Run **one** shared workflow as a team. One person drives the agent and the others take part in every decision.
- Start with: *"Using AI-DLC, I want to start a new project. Please read `docs/founding-idea.md` and `docs/technical-environment.md`, then begin the AI-DLC workflow."*
- Answer every question deliberately. Push back and request changes when a proposal doesn't fit, because graders look for engagement beyond approvals.
- Reset the agent's context at each gate.
- Start exploratory questions with "Do not update any documents." Then ask the agent to log them in `audit.md`.
- Never hand-edit AI-DLC artifacts or `audit.md`. Fix problems through the workflow.
- Commit `audit.md` and `aidlc-state.md` along with the artifacts.

## 6. Before submitting

- [ ] Everything is committed and pushed
- [ ] The repo URL opens in a private or incognito window without signing in
- [ ] The URL is submitted on Canvas

## 7. Reference links

**AI-DLC v1.0.1** (use these, not `main`, which is v2):

- [Release and rules zip](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1)
- [Setup README](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/README.md)
- [Working with AI-DLC](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/docs/WORKING-WITH-AIDLC.md)
- [Inputs quickstart](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/docs/writing-inputs/inputs-quickstart.md)
- [Example vision document](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/docs/writing-inputs/example-minimal-vision-scientific-calculator-api.md)
- [Example technical environment document](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/docs/writing-inputs/example-minimal-tech-env-scientific-calculator-api.md)
- [Generated docs reference](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/docs/GENERATED_DOCS_REFERENCE.md)
- [Methodology (AWS DevOps Blog)](https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/)

**LLM options inside the product:**

- [Ollama download](https://ollama.com/download) · [Model library](https://ollama.com/library) · [Quickstart](https://docs.ollama.com/quickstart) · [OpenAI-compatible API](https://docs.ollama.com/openai)

**Course:**

- [Canvas assignment page](https://canvas.cornell.edu/courses/91792/assignments/1036177)
- Final Project, due Dec 16: Construction for every unit, a demo video, a recorded presentation, and peer evaluation
