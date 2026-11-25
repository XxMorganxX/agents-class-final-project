# Application Design Clarification Questions

**Stage:** INCEPTION - Application Design (Step 9: follow-up questions)
**Source:** answers in `application-design-plan.md` (filled 2026-11-25)

I checked the 15 answers against each other and against requirements.md and stories.md. One answer conflicts with an approved requirement, and the team has to decide which one gives way. The other points I found can be settled within your answers; they are listed under "Answer analysis" in `application-design-plan.md` for you to check.

## Contradiction 1: where the per-session token lives

You chose **Q11 B**: the hook entry runs `node <script>`, and the script reads the token and port from a file Pi writes at startup. The entries in `settings.json` then never change between sessions.

Two approved items say the token is in the hook command itself:
- **NFR-5.3:** "It accepts only events carrying the random per-session token that Pi puts in the hook command."
- **US-04:** "[test] The hook command carries a per-session token (US-06)."

Q11 B keeps the security goal (only a caller that knows this session's random token can post events), but the token sits in a runtime file in Pi's app-data folder, readable only by the user (mode `0600`), not in the command. Putting it in the command means rewriting `~/.claude/settings.json` at every launch, which works against FR-1.7 and AC-6 (removing Pi's hooks leaves the file exactly as it was before install).

### Clarification Question 1
How should the design resolve this?

A) Keep Q11 B. The token lives in a `0600` runtime file in Pi's app-data folder, deleted when Pi quits. The design records this as a change to the wording of NFR-5.3 and US-04 ("the hook script sends a per-session token that Pi writes at startup"), noted in requirements §3 style, and the US-04 test checks the script and runtime file instead of the command

B) Follow NFR-5.3 as written: keep the Node script from Q11 B, but Pi rewrites its own hook entries at each launch to pass the new token as a command argument. Install still asks consent once; later rewrites touch only Pi's entries

C) Other (please describe after [Answer]: tag below)

[Answer]: A: keep Q11 B and reword NFR-5.3 and US-04 to match. Condition: Pi writes the token file with owner-only permissions (0600) so other users on the machine can't read it, and Pi rotates the token every launch like before.
