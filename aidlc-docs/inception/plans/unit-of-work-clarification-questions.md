# Units Generation Clarification Questions

**Stage:** INCEPTION - Units Generation (Part 1, Step 4)
**Source:** team answers Q1-Q15 in `unit-of-work-plan.md`

I found two contradictions and one gap in the answers. Please answer each question below by writing the letter after its `[Answer]:` tag. If no option fits, choose the last option (Other) and describe what you want.

## Contradiction 1: where U1's contract interfaces live

Q4 puts the interfaces of `TutorModel`, `BuilderBridge` and `VideoRunner` in U1 (Foundation). Q1 says each unit owns clear folders, and Q14 says "no design changes". The approved source layout puts those interfaces inside other units' folders:
- `TutorModel` is in C12, in `src/main/tutor/` (U3)
- `BuilderBridge` is in C14, in `src/main/builder-bridge/` (U3)
- `VideoRunner` is in C21, in `src/plugins/video/` (U7). components.md says C21 "holds the Video Runner contract"

So U1 can't own these interfaces without either sharing folders with U3 and U7, or moving the files, which changes the design.

### Clarification Question 1
Where should the three interfaces that U1 owns live?

A) In their approved folders, each in its own file: `src/main/tutor/tutor-model.ts`, `src/main/builder-bridge/builder-bridge.ts`, `src/plugins/video/video-runner.ts`. U1 owns those three files, and U3 and U7 own the rest of their folders. `unit-of-work.md` lists folder ownership down to these files. No design change

B) In a new folder, `src/main/contracts/`, owned by U1. This changes the design: a new folder in the source layout in components.md, and the `VideoRunner` contract moves out of C21 (components.md, component-methods.md). I would make those edits and report them. The core still never imports `src/plugins/video/` (D14), because the plug-in would import the contract, not the other way round

C) U1 holds only `TutorModel` and `BuilderBridge` (option A placement). `VideoRunner` and `FakeVideoRunner` stay entirely in U7. U7 also adds the `videoRunner` field to `AppOverrides` in the composition root when it is built. No design change, but U1's `createApp` would have no video override until U7

D) Other (please describe after [Answer]: tag below)

[Answer]: C. U1 holds the TutorModel and BuilderBridge interfaces in their approved files (src/main/tutor/tutor-model.ts and src/main/builder-bridge/builder-bridge.ts). VideoRunner and FakeVideoRunner stay entirely in U7. Video is optional and may be dropped at the Dec 10 cutoff, so keeping all of it in U7 means dropping it touches no other unit. No design change.

## Contradiction 2: "runs in the real app" before the app can run

Q11 C says a unit is done only when it also runs in the real app through `createApp` (Q6 A: as soon as two units are done). The Q5 order is U1, U3, U4, U2, U6, U5, U7. When U3 and U4 finish, the app has no Session Orchestrator (U5) to start agent runs, no Hook Server (U2) to receive events, and no window (U6) to show cards. So U3 and U4 can't run in the real app at the time they would be marked done.

### Clarification Question 2
How should the "runs in the real app" check work for units built before the app can run end to end?

A) Check through the nearest working entry point. `createApp` builds the graph with the new unit wired in, and an integration test drives it through the closest existing entry. For U3, a recorded hook fixture goes straight to the Tutor Agent. For U4, the cards go through the registry. When U5 joins, it repeats the full-app check for U3 and U4. U3 and U4 count as done after their own check

B) Change the order so the app runs end to end sooner: U1, U3, U4, U5, U2, U6, U7. U5 joins the agent and the card types early (driven by fixtures with a recording `FeedSink`), and U2 and U6 plug into a core that already runs. U3 and U4 stay first, as in Q5

C) Keep the order. U3 and U4 are marked "done pending the app check", and that check closes when U5 joins. Any problem found then reopens the unit

D) Other (please describe after [Answer]: tag below)

[Answer]: B, with A's check for the units built before U5. Final build order: U1, U4, U3, U5, U2, U6, U7 (CQ3 explains U4 before U3). U4 and U3 are checked through their nearest working entry point when they finish: the registry for U4, and a recorded hook fixture into the Tutor Agent for U3. U5 comes right after them, so the full-app check for both runs one unit later, and a failure there reopens whichever unit caused it. Moving U5 up gets the core teaching loop running end to end by the fourth unit, and the hooks and window then plug into a core that already works.

## Gap 1: U3's live answer-key check needs U4's card types

Q5 builds U3 before U4, and Q11 C gives U3 a live check against Haiku because of AC-4. AC-4 is a hand review of 20 **exercises** generated from recorded hook fixtures, and 90% must have a correct answer key. The exercise types (multiple-choice, written-response, quick-time, prediction) belong to U4 (Q3 A). Until U4 is built, the agent's `show_component` union has no exercise types, so the AC-4 review can't run when U3 finishes.

### Clarification Question 3
How should U3's live check relate to AC-4?

A) U3's live check confirms that real Haiku runs on recorded fixtures produce valid `show_component` calls and that `claude -p` returns verdicts, using one test exercise type registered only in the test. The full AC-4 review (20 exercises, 90%) runs at the end of U4, and a failure there reopens U3 or U4, whichever caused it

B) U3 is not done until the full AC-4 review passes, which happens after U4 is built. U4's loop finishes both units

C) Swap the order to U4 then U3, so the exercise types exist and U3's live check can be the full AC-4 review

D) Other (please describe after [Answer]: tag below)

[Answer]: C. Build U4 before U3. Then the exercise types exist when U3 is built, so U3's live check can be the full AC-4 review (20 exercises, 90% correct) instead of a test-only stand-in. Both units still come right after U1, so the answer-key risk is still handled early.

---

# Round 2 (after Clarifications 1-3)

The answers to Clarifications 1-3 agree with each other and with Q1-Q15. The new build order (U1, U4, U3, U5, U2, U6, U7) opens two gaps, because two units now need code from units built after them.

## Gap 2: U4's live grader check needs the real model, which is U3 code

Reading R1 (accepted) gives U4 a live check, because the written-response grader (and prediction judging, if Functional Design needs it) calls `TutorModel.generateJson`. After Clarification 1, U1 holds only the `TutorModel` interface. The real `PiTutorModel` and the `BudgetedTutorModel` decorator are C12, which is in U3. U4 is now built before U3, so U4 has no real model to call when it finishes.

### Clarification Question 4
How should U4's live grader check work?

A) Move it into U3's loop. U4 is done with fakes and its registry check. When U3 builds `PiTutorModel`, U3's live check also grades a set of real written responses through U4's grader against real Haiku. A failure reopens U4

B) Move `PiTutorModel` and `BudgetedTutorModel` (C12's real implementations) into U1, so U4 can make its live check. U1 then holds a real model client, not only the interface, and U3 keeps C10, C11, C13 and C14

C) Drop the live check for U4. Checklist grading quality is checked in U5's full-app check and in the AC-3 end-to-end run at Build and Test

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Move U4's live grader check into U3's loop, and a failure there reopens U4. U3 builds the real PiTutorModel and already runs the full AC-4 review against Haiku, so both live grading checks run where the real model first exists. This keeps Q4's rule that U1 holds interfaces, not implementations.

## Gap 3: U3 and U5 need the fixture-to-trigger code, which is U2 code

Clarification 2 checks U3 with "a recorded hook fixture into the Tutor Agent", and AC-4 uses "exercises generated from recorded hook fixtures". A recorded fixture is a raw Claude Code hook payload. Turning it into the agent's `builder-activity` trigger takes C8 Builder Events: `toBuilderEvent`, `mergeActivity` and `readTurnDetails` (for the end-of-turn message and test runs). C8 is in U2 (draft), which is now built fifth, after U3 and U5. The `BuilderEvent` and `Trigger` types are already shared (C2, in U1), so only C8's logic is missing.

### Clarification Question 5
Where should C8 Builder Events go?

A) Move C8 from U2 to U1. Its logic is pure apart from one transcript read, so it suits the foundation. U3 and U5 can then run real fixtures end to end. U2 keeps C5 Hook Server, C6 Hook Installer and C7 Setup Checks. This is a change to unit grouping only, not to the design

B) Move C8 from U2 to U3, the first unit that needs it. U5 then uses it through U3. U2 keeps C5, C6 and C7

C) Keep C8 in U2. Until U2 is built, U3 and U5 tests use triggers written by hand next to each recorded fixture, and the AC-4 review is run again on the real C8 output once U2 lands

D) Other (please describe after [Answer]: tag below)

[Answer]: A. Move C8 Builder Events to U1. It is a pure translator from hook payloads to agent triggers, and every unit downstream of the hooks needs it (U3's fixture check, U5, U2). Having it in the foundation also means the AC-4 review runs on real recorded hook fixtures from the start, not hand-written triggers. US-06 and US-03 stay in U2, with U1 as a supporting unit.

---

# Round 3 (during the review of the unit files)

The team added a rule to the story map: if U7 is dropped, the video-only criteria of core stories are recorded as not applicable, and the stories close without them. Applying it showed one more place that depends on U7 but isn't a story, so the rule doesn't cover it.

## Gap 4: AC-5 and journey J2 need U7's requirement check

- **AC-5** (requirements.md): "a clean machine without the video runner runs the full demo flow, **and turning videos on shows what is missing**."
- **J2 step 5** (stories.md): "Sam turns videos on. Pi names the missing requirement and leaves videos off."

Checking for the missing pieces is `checkRequirements` in C21, which is U7 code (US-V1). If U7 is dropped at the 2026-12-10 cutoff, nothing can show what is missing, so the second half of AC-5 and J2 step 5 can't pass. The first half of AC-5 (the demo runs without the video runner) still passes.

### Clarification Question 6
If U7 is dropped, what happens to AC-5's "turning videos on shows what is missing" and J2 step 5?

A) Apply the same rule. Both are recorded as not applicable and closed without them, and the decision is logged in `audit.md`. This changes how a requirement is met, so requirements.md (AC-5) and stories.md (J2) each get a dated note saying the video part doesn't apply when the video unit isn't built

B) Keep AC-5 whole. Split US-V1's requirement check out of U7 so it is built even if U7 is dropped: C21's `checkRequirements` and the settings message that names what is missing, with videos staying off. It runs as a small loop right after U6 and before the cutoff decision. US-V2 and US-V3 stay in U7. No requirement change, but one more loop and unit files that change (the U7 split, the build order and the story map)

C) Other (please describe after [Answer]: tag below)

[Answer]: A. Apply the same rule to AC-5 and J2, with dated notes in requirements.md and stories.md. If U7 is dropped, Pi has no video setting at all, so "turning videos on shows what is missing" has nothing to test. The first half of AC-5, a clean machine without the video runner running the full demo flow, stays fully required. That's the part our stories called our biggest demo risk. Option B would add another loop to a schedule that is already tight (R5).
