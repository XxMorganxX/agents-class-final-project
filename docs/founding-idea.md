# Anti-Vibe Coding: An Interactive Learning Sidecar for AI Coding Agents

**Status:** Product concept / initial architecture

## 1. Vision

AI coding agents have dramatically increased the amount of software a developer can produce, but they have also created a new problem: developers can build increasingly sophisticated systems without understanding how those systems work.

We want to reverse this trend.

Our goal is to create an **interactive learning sidecar that turns every AI-assisted coding session into an opportunity to develop genuine technical understanding.**

The system uses two complementary agents:

- **Claude Code — The Builder:** The primary software engineering agent responsible for writing code, debugging, architecture, and solving complex engineering problems.
- **Pi — The Educator:** A lightweight, persistent companion responsible for tracking the user's understanding, creating interactive learning experiences, and determining when additional instruction is needed.

As Claude Code develops software, Pi observes the changes, identifies important concepts, and dynamically generates educational interactions that test whether the developer understands what is being built.

Rather than replacing AI-assisted development, we want to make it a mechanism for accelerating human learning.

**Core principle: The developer should become more technically capable with every coding session, not more dependent on the agent.**

### Target users

Our primary users are **students learning to code with AI coding agents**: for example, computer science students who use Claude Code for coursework and personal projects. These students are still building the fundamentals that an agent lets them skip. They are the most likely to ship code they can't explain, and they gain the most from turning each session into practice. "Developer" in this document means this student user unless stated otherwise.

## 2. The Problem

Traditional software development naturally required developers to understand much of the code they produced. AI coding agents weaken this relationship.

A developer can now ask an agent to build an authentication system, database integration, or distributed architecture without understanding the implementation, its tradeoffs, or its failure modes.

This creates several problems:

- **Illusion of understanding:** Developers recognize what their code does but cannot explain why it works.
- **Architectural dependency:** Developers rely on agents to make design decisions they cannot independently evaluate.
- **Lost learning opportunities:** Complex problems are solved automatically instead of becoming opportunities to develop engineering intuition.
- **Weak debugging intuition:** Developers struggle to reason about failures without delegating the entire investigation back to the agent.
- **Skill degradation:** AI increases immediate productivity without necessarily increasing long-term engineering capability.

We want to preserve the productivity benefits of coding agents while creating a system that actively develops the knowledge necessary to use them effectively.

## 3. Core Product Experience

The system operates continuously alongside an ordinary Claude Code development session.

The developer continues working with Claude Code as usual. Meanwhile, Pi maintains awareness of what Claude is building and periodically introduces interactive educational experiences.

These experiences are generated from the actual codebase, current task, and developer's demonstrated understanding.

### A. Live Code Explanations

As Claude Code writes or modifies software, Pi identifies meaningful implementation decisions and explains them in context.

Examples:

- Why Claude introduced a particular abstraction.
- How data flows between two newly created components.
- Why one concurrency model was selected over another.
- What a new API endpoint actually does.
- How a database schema supports the requested functionality.

Explanations should be selective, concise, and grounded in the actual implementation.

The system should prioritize concepts that are important, unfamiliar, or likely to affect the developer's future decisions.

### B. Interactive Multiple-Choice Quizzes

Pi generates targeted questions that test understanding of the software currently being built.

For example, after Claude implements a caching layer:

**Question:** What happens if two requests access the same uncached resource simultaneously?

A. Both requests are automatically serialized.

B. Both may independently compute and populate the cache.

C. The second request always receives a cache hit.

D. The cache rejects concurrent requests.

The correct answer depends on the actual implementation.

Pi evaluates the user's response, explains any misconception, and updates its understanding of the user's knowledge.

Questions should progressively move from code recognition toward predicting behavior and evaluating engineering tradeoffs.

### C. Short-Response and Teach-Back Exercises

Some concepts require more than recognizing the correct answer.

Pi can ask the user to explain a behavior in their own words:

- "Walk me through what happens when this endpoint receives a request."
- "Why did Claude split this functionality into two services?"
- "What could go wrong if this operation fails halfway through?"
- "Explain how this function prevents a race condition."

The system evaluates conceptual understanding rather than exact wording.

Incorrect answers trigger focused follow-up questions instead of immediately revealing the solution.

### D. AI-Generated Educational Videos

We already have access to a video generation engine, which should become a central instructional tool.

Pi can turn particularly important or difficult concepts into short, contextual videos.

Potential formats include:

- **Animated execution traces:** Visualize data moving through the actual application.
- **Architecture walkthroughs:** Explain how services, modules, and dependencies interact.
- **Algorithm animations:** Show how a newly implemented algorithm processes inputs.
- **Failure simulations:** Illustrate race conditions, deadlocks, unexpected states, or other bugs.
- **Conceptual explanations:** Introduce unfamiliar computer science concepts relevant to the implementation.

Videos should be short and generated selectively based on conceptual difficulty, instructional value, and the user's demonstrated need.

The ideal experience is that a developer struggling with a complex function can receive a visual explanation tailored to that exact function, rather than searching for a generic tutorial.

Video generation should be asynchronous so that the coding session continues uninterrupted.

### E. Quick-Time Events

Pi can occasionally interrupt the developer with a short, time-sensitive challenge related to the current implementation.

Examples:

**Predict the execution:** Claude has written an asynchronous function. Pi presents the execution order and asks the developer to predict which operation completes first.

**Spot the bug:** Pi shows a small piece of code and asks the developer to identify a failure condition.

**Choose the architecture:** Before Claude implements a new component, Pi asks the developer to select between competing approaches.

**Predict the output:** Pi presents an input to a recently created function and asks the developer to predict the result.

These interactions should be lightweight, engaging, and highly contextual.

Timed challenges are optional and should only be used when speed is relevant. The primary goal is reasoning, not reaction time.

### F. System Design and Pseudocode Exercises

For higher-level engineering concepts, Pi asks developers to actively design parts of the system themselves.

Before or after Claude implements a major feature, Pi can present a simplified design exercise.

For example:

"Claude is about to implement a job queue for processing background tasks. Describe in pseudocode how you would ensure a failed task can be retried without being processed twice."

The user can write pseudocode, draw a component architecture, or explain a design decision.

Pi evaluates the solution against the actual requirements and implementation, providing feedback on correctness, assumptions, tradeoffs, and missing edge cases.

These exercises are particularly important because they require developers to demonstrate the ability to construct solutions rather than merely understand existing ones.

## 4. Adaptive Learning and Knowledge Tracking

Pi should maintain a continuously evolving model of what the developer understands.

This model should distinguish between:

- Concepts the developer has encountered.
- Concepts the developer can recognize.
- Concepts the developer can explain.
- Concepts the developer can apply to new problems.
- Concepts the developer repeatedly misunderstands.

For each concept, Pi maintains an estimate of mastery, uncertainty, and supporting evidence.

**Example knowledge profile:**

| Concept | Observed understanding | Next action |
|---|---|---|
| REST APIs | Strong | Skip basic explanations |
| Async execution | Developing | Prediction exercise |
| Database indexing | Weak | Visual explanation |
| Race conditions | Unknown | Diagnostic question |
| System design | Strong | Advanced tradeoff exercise |

When the developer answers incorrectly, Pi should diagnose the specific misconception and drill down.

For example:

1. A developer incorrectly predicts the behavior of a concurrent function.
2. Pi determines whether the misunderstanding concerns promises, scheduling, shared state, or synchronization.
3. Pi generates a simpler question targeting the likely misconception.
4. If necessary, Pi requests a visual explanation from the video engine.
5. Pi retests the original concept with a different example.
6. Pi revisits the concept in a future session to test retention.

The goal is to make the learning experience adaptive rather than a random series of quizzes.

## 5. Multi-Agent Architecture

The architecture should preserve a strict separation between software engineering and educational orchestration.

### Claude Code: Primary Engineering Agent

Claude Code remains responsible for:

- Implementing features.
- Reading and modifying the codebase.
- Debugging and testing.
- Making complex architectural decisions.
- Answering difficult technical questions delegated by Pi.

Claude Code is the expensive, high-capability reasoning engine.

### Pi: Lightweight Educational Sidecar

Pi is responsible for:

- Monitoring development events.
- Maintaining learning state.
- Deciding when an intervention is valuable.
- Selecting the appropriate educational format.
- Generating straightforward questions and explanations.
- Evaluating simple responses.
- Coordinating the video generation engine.
- Requesting deeper technical context from Claude when necessary.
- Presenting the dynamic learning interface.

Pi should use inexpensive models and deterministic logic wherever possible.

### Communication Between Agents

Communication should be bidirectional.

**Claude Code → Pi**

Claude provides information about:

- Current task and objectives.
- Files and functions changed.
- Significant architectural decisions.
- Implemented algorithms and their intended behavior.
- Relevant tests and execution results.
- Concepts requiring deeper understanding.

**Pi → Claude Code**

Pi can request:

- Clarification of a difficult implementation.
- Explanation of a technical decision.
- Identification of important concepts worth testing.
- Verification of an educational question's answer.
- Analysis of a developer's proposed architecture.
- Additional context needed to produce an accurate video.

Pi should avoid duplicating complex reasoning that Claude Code has already performed.

The communication layer should be asynchronous, with event-driven updates and targeted requests rather than continuous conversational polling.

## 6. Dynamic User Interface

The educational experience should be presented through a dynamic interface controlled by Pi.

The UI is not a static chat window. It adapts to the concept being taught and the form of interaction that would best test understanding.

Potential interface components include:

- Interactive multiple-choice cards.
- Code viewers with highlighted execution paths.
- Short-answer fields.
- Pseudocode editors.
- Architecture diagrams.
- Embedded generated videos.
- Timed prediction challenges.
- Progress and concept-mastery visualizations.

The interface should support different levels of interruption.

**Passive mode:** Pi explains important implementation decisions without demanding attention.

**Interactive mode:** Pi periodically introduces questions and short exercises.

**Deep learning mode:** Pi pauses the educational sequence at major milestones and guides the developer through more comprehensive assessments.

The central design challenge is integrating learning without destroying the developer's productive flow.

## 7. Example End-to-End Workflow

**Scenario: A developer asks Claude Code to implement a real-time WebSocket messaging system.**

**Step 1 — Development begins**

Claude starts implementing WebSocket connections, message broadcasting, and connection state management.

Pi observes the implementation and identifies concurrency, connection lifecycle, and event-driven communication as relevant concepts.

**Step 2 — Initial understanding check**

Pi asks:

"What is the primary difference between a WebSocket connection and a traditional HTTP request?"

The user answers correctly. Pi records the result and avoids unnecessary introductory explanations.

**Step 3 — Implementation-specific challenge**

Claude implements a mapping from user IDs to active socket connections.

Pi asks:

"What happens when a single user opens the application in two browser tabs?"

The user gives an incorrect answer.

**Step 4 — Targeted remediation**

Pi identifies that the user has misunderstood the relationship between user identity and connection identity.

Pi requests a concise explanation from Claude about the actual data structure and its behavior.

**Step 5 — Visual instruction**

Pi uses the video engine to generate a short animation showing two browser tabs establishing separate connections and interacting with the server's connection map.

**Step 6 — Retest**

Pi presents a new scenario involving three connections, one disconnect, and an incoming broadcast.

The user predicts which clients receive the message.

**Step 7 — Design exercise**

Once the basic concepts are understood, Pi asks:

"Write pseudocode for a connection manager that supports multiple connections per user and cleans up disconnected clients."

Pi evaluates the response and provides architectural feedback.

**Step 8 — Learning state update**

Pi records the user's demonstrated understanding and schedules a related question for a future session.

Throughout the process, Claude Code remains the primary development agent. Pi provides the educational layer without taking over the engineering workflow.

## 8. Cost and Efficiency Principles

Keeping Pi inexpensive is a core architectural requirement.

The system should follow these principles:

**1. Reuse Claude's existing reasoning.** Pi should consume concise summaries and implementation metadata instead of independently reanalyzing the entire codebase.

**2. Use event-driven processing.** Trigger educational analysis after meaningful changes, such as a completed feature, significant function modification, or architectural decision, rather than every token or tool call.

**3. Use a tiered model strategy.** Simple question generation, classification, and learning-state updates should use inexpensive models. Difficult technical reasoning should be escalated to Claude.

**4. Keep context minimal.** Pi should receive the relevant code snippets, diffs, summaries, and learning records rather than repeatedly loading the full repository.

**5. Cache educational materials.** Reuse explanations, videos, and questions when the underlying concepts and implementations have not materially changed.

**6. Generate videos selectively.** Video generation should be reserved for concepts that benefit from visual instruction, especially when simpler explanations have not worked.

**7. Avoid unnecessary assessments.** Pi should optimize for actual learning gains, not the number of questions or tokens generated.

**Design objective:** Maximize demonstrated learning per dollar spent while introducing minimal additional latency to the primary coding workflow.

## 9. Technical Design Considerations

### Session Synchronization

Pi needs a reliable understanding of Claude's current work.

A shared session layer should associate a development task with its relevant code changes, decisions, test results, and educational activities.

Information passed between agents should include file references, code versions, and enough context to ensure that educational content reflects the current implementation.

### Grounding and Correctness

Educational questions must be based on the real code rather than generic assumptions.

A correct answer in one implementation may be incorrect in another.

For complex questions, Pi should obtain a verified answer from Claude or use relevant tests and execution traces.

If the underlying implementation changes, existing questions and explanations may need to be invalidated.

### Noninterference

Pi should be read-only with respect to the production codebase by default.

Educational activities should not interrupt Claude's tool execution, corrupt files, or introduce competing modifications.

Any programming exercises that require code execution should run in an isolated environment.

### Evaluation

Answer evaluation should distinguish:

- Factual correctness.
- Quality of reasoning.
- Transfer of knowledge to new situations.
- Completeness of architectural considerations.
- Confidence and uncertainty.

The system should not infer mastery from a single correct multiple-choice answer.

## 10. MVP Scope

The MVP is what we will build for the course's Final Project, due December 16, 2026. Where earlier sections describe the long-term product, this section sets what is actually in scope.

**MVP statement:** Pi is a coding sidecar that runs beside Claude Code and talks to it in both directions. Its job is to check that the developer understands the code being built. Pi is a dynamic UI that explains code while Claude writes and runs it, and it tests understanding with several interaction types. Pi stays cheap, and Claude Code does the hard technical reasoning.

### In scope

- **One Pi session paired with one Claude Code session**, with two-way communication:
  - Claude → Pi: the task, changed files and functions, key decisions, and test results.
  - Pi → Claude: requests for explanations, answer verification, and concept identification.
- **Live code explanations** grounded in the actual implementation while Claude works (§3A).
- **Multiple-choice quizzes** generated from the current code (§3B).
- **Short-response and teach-back exercises**, graded on understanding rather than exact wording (§3C).
- **Generated explanation videos**, using our existing video generation engine to explain important concepts as the code is built (§3D). Videos are generated asynchronously so the coding session isn't blocked.
- **Quick-time events**, such as predicting execution, spotting a bug, choosing an architecture, or predicting output (§3E).
- **Pseudocode and system design exercises**, with feedback on correctness, assumptions and missed edge cases (§3F).
- **Drill-down on wrong answers.** When the developer answers wrong, Pi asks focused follow-up questions on the same concept to find the misconception, then retests with a new example (§4).
- **Session-scoped learning state.** Pi records which concepts came up and how the developer answered during the current session.
- **Cost-conscious design.** Pi uses inexpensive models and deterministic logic, and escalates hard reasoning to Claude Code (§8).

### Out of scope

- **Difficulty scaling.** Pi does not automatically raise or lower question difficulty based on the developer's track record. Drill-down stays at the level of the concept that was missed.
- **Remediation videos tailored to misunderstanding.** Videos explain concepts as the code is built. In the MVP they are not generated in response to a specific wrong answer.
- **Detailed models of the developer's prior education.** Pi does not build a long-term profile of what the developer already knows from earlier courses, jobs or projects. Learning state lasts for one session.
- **Everything listed under Future versions below.**

### Future versions

These ideas fit the product but are left out of the MVP.

**v2: Adaptive learning**

- Difficulty scaling based on measured mastery.
- Remediation videos tailored to the developer's specific misconception.
- A persistent knowledge profile across sessions, as described in §4, with spaced review of concepts the developer struggled with.
- A concept-mastery dashboard showing progress over time.
- Caching and reuse of questions, explanations and videos when the underlying code hasn't changed (§8.5).
- Before-and-after comprehension checks to measure learning gains (§11).
- Configurable interruption levels (passive, interactive, deep learning) per session (§6).

**v3: Learning platform**

- A personalized curriculum based on the developer's prior education and experience.
- Support for other coding agents besides Claude Code, such as Codex, Cursor and Copilot, behind a shared event interface.
- An instructor or team-lead mode, where a course or team sets the concepts to cover and reviews learners' progress.
- Sandboxed code-execution exercises, where the developer edits and runs code in isolation from the real codebase.
- An onboarding mode that walks a new contributor through an existing codebase rather than new code.
- A pull-request review mode that quizzes a reviewer on a teammate's changes before approval.
- Voice-based teach-back exercises.
- Intervention policies tuned from measured outcomes, deciding when and how to teach for the most learning per dollar.

## 11. Success Metrics

The product should not be evaluated primarily by how much educational material it generates.

The most important metric is whether developers become better at reasoning about their software independently.

Potential measurements include:

- **Comprehension:** Can the developer accurately explain newly implemented code?
- **Prediction:** Can the developer predict program behavior in unfamiliar scenarios?
- **Transfer:** Can the developer apply a concept to a different implementation?
- **Retention:** Can the developer demonstrate understanding days or weeks later?
- **Independence:** Can the developer solve related problems with less agent assistance?
- **Interruption cost:** How much does the educational experience slow or disrupt development?
- **Efficiency:** How much learning improvement is achieved per dollar of inference and video generation cost?

The ideal outcome is increased developer competence without materially reducing the productivity advantages of Claude Code.

## 12. Long-Term Direction

The broader opportunity is to create a learning layer for agentic software development.

Today, coding agents optimize for producing working code. Our system would additionally optimize for increasing the user's ability to understand, evaluate, and eventually design that code independently.

Over time, the sidecar could develop an increasingly accurate model of a developer's technical abilities, weaknesses, and interests.

This would allow AI-assisted development to become a personalized, continuous software engineering education.

The ultimate goal is not to make developers write every line of code themselves.

**It is to ensure that, even as agents write more of the code, the developers directing them continue to become better engineers.**

## 13. Open Questions

1. **Interface: web app or terminal?** Should Pi be a web app beside the terminal, or run inside the terminal next to Claude Code? A web app makes rich components such as videos, diagrams and code viewers easier. A terminal interface keeps the developer in one place.
2. **Blocking the Builder:** Can we lock the Claude Code session and make it wait until the developer finishes a Pi session? If so, when should Pi block Claude, and when should learning run alongside it without blocking?
3. **Pi–Claude communication:** How does communication between the agents actually work? This covers how Pi learns what Claude changed and how Pi sends questions back to Claude for verification or explanation.
4. **Interaction types:** What is the full set of interaction types inside Pi? The MVP includes all of §3: explanations, multiple-choice questions, short-answer and teach-back exercises, videos, quick-time events, and pseudocode or system design exercises. Are there others, and how does each one appear in the UI?
