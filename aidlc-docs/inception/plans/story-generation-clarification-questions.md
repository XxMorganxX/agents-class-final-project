# Story Generation Clarification Questions

**Stage:** INCEPTION - User Stories (Part 1: Planning, Step 10 follow-up)
**About:** the team's answers in `story-generation-plan.md` (written in 2026-11-24)

The 16 answers don't contradict each other or requirements.md. Four of them leave a decision open that would change how the stories are written. Write a letter after each `[Answer]:` tag. Option A is the agent's recommendation in each case.

---

## Question 1
Q1 adds an intro-level student persona. Requirements §4 says Pi pitches every question at the intermediate level and defines each technical term the first time it uses it. Difficulty scaling is v2 (§8). So how should stories serve the intro student?

A) Through what is already in scope: first-use term definitions, the "explain first" path after a miss (FR-3.2), drill-down (FR-4.3) and skip (FR-3.5). The intro persona tests whether those are enough. No story adjusts difficulty per student (Recommended)

B) Add acceptance criteria that adjust wording or difficulty for intro students. This reopens a v2 item, and requirements.md would need a change first

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Team accepted the recommendation (2026-11-24).

## Question 2
Q12 (C) and Q13 (B) overlap. Pi's read-only tools, its step limit and its Builder calls could be written as student outcomes ("Pi never touches my repo"), as "As Pi..." stories, or as both. Which rule decides?

A) "As Pi..." stories cover the agent's **decisions**: what to teach and when, which component to pick, when to show nothing, when to ask the Builder and when to drop a question. **Limits** on the agent (read-only tools, repo boundary, secret files, step limit, caps) are student-facing stories per Q12, written as trust and cost outcomes. Team stories cover only logging and the config module (Recommended)

B) All FR-0 items, decisions and limits, go into the "As Pi..." stories. Student-facing outcomes cover only the hooks and the Electron window

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Team accepted the recommendation (2026-11-24).

## Question 3
Q9 (C) allows **[review]** criteria only on stories about lesson content and grading. Does that include the "As Pi..." stories from Q13? Whether Pi picked a concept worth teaching at that moment is a quality judgment, so no automated test can decide it.

A) Yes. The "As Pi..." decision stories count as lesson content and may carry **[review]** criteria for relevance. Their limits (one component per run, nothing shown when no listed concept applies, the step limit) stay **[test]** (Recommended)

B) No. "As Pi..." stories get **[test]** criteria only. Decision quality is judged only through AC-4's hand review

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Team accepted the recommendation (2026-11-24).

## Question 4
Q5 sets 25 to 35 stories. Q6 (5 component stories), Q13 (the "As Pi..." stories), Q15 (full video stories) and Q12 (team stories) all add to that count. The agent's estimate is 33 to 38. What should happen if the count goes over 35?

A) Keep 25 to 35 as the target, but don't count the video stories (tagged Optional feature). Merge stories only if the core set, without video, goes over 35 (Recommended)

B) 35 is a hard cap across all stories, video included. Merge stories until the total fits

C) Treat the range as a guide only. Write as many medium-sized stories as the requirements need

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Team accepted the recommendation (2026-11-24).
