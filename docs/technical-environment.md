# Technical Environment: Anti-Vibe Coding (Pi)

This document gives AI-DLC the technical constraints for building Pi, the learning sidecar described in [`founding-idea.md`](founding-idea.md). It covers the MVP in §10 of that document.

## 1. System at a glance

Pi is one Electron desktop app running beside Claude Code on the same laptop:

```
+------------------+   hook events (HTTP)    +-------------------------------+
|   Claude Code    | ----------------------> |  Pi app: Electron main        |
|  (the Builder)   |                         |  - hook server 127.0.0.1:4780 |
|  student's repo  | <---------------------- |  - pi agent harness (SDK)     |
+------------------+  read-only `claude -p`  |  - video job queue            |
                         questions           +-------------------------------+
                                                   |                 |
                                    IPC via preload|                 | subprocess
                                                   v                 v
                                 +-------------------------+  +------------------+
                                 | Pi app: renderer        |  | make-video       |
                                 | chat window (React)     |  | (Project-Feynman)|
                                 +-------------------------+  +------------------+
```

- **Claude Code → Pi.** Claude Code [hooks](https://docs.claude.com/en/docs/claude-code/hooks) post a JSON event to Pi's local server after each file edit, when a turn ends, and when the student sends a prompt.
- **Pi → Claude Code.** Pi asks the Builder questions through a separate, read-only headless call (`claude -p`) in the student's repo. It never types into the student's live session.
- **Pi → student.** The Electron main process runs the pi harness and sends UI cards to the chat window over Electron IPC: explanations, quizzes, quick-time events, pseudocode exercises and videos. The student's answers come back the same way.
- **Pi → video engine.** Pi starts Project-Feynman's `make-video` CLI as a background subprocess and shows the MP4 when the render finishes.

## 2. Language, runtime, package manager and frameworks

| Area | Choice | Why |
|---|---|---|
| Language | TypeScript 5.x, `strict: true` | Pi's agent harness is a TypeScript SDK. |
| Runtime | Node.js 24 LTS (at least 22.19, which the pi SDK requires) | Required by the pi SDK. |
| Package manager | npm with a committed `package-lock.json`. Run `npm ci --ignore-scripts`. | Matches how the pi harness is pinned elsewhere. Lifecycle scripts stay off. |
| Agent harness | [`@earendil-works/pi-coding-agent`](https://www.npmjs.com/package/@earendil-works/pi-coding-agent) 1.4.0 (pinned) SDK | Gives Pi its agent loop, sessions, model runtime and custom tools (`pi.registerTool`). |
| Model (Pi) | Claude Haiku 5.5 (`claude-haiku-5-5`) through pi's `anthropic` provider | Cheap and fast enough for question generation and grading. Swappable through pi's model runtime. |
| Model (escalation) | Claude Code headless (`claude -p`), using the model on the student's Claude Code login | Hard reasoning goes to the Builder, as the founding idea requires (§5, §8). |
| Desktop app | Electron (latest stable at project start, pinned exactly) with Electron Forge and its Vite plugin | Gives a dedicated chat window built for learning, with native video playback. Forge is Electron's official build tool. |
| Hook server | Fastify 5, inside the Electron main process | Claude Code hooks need an HTTP endpoint. Fastify gives typed routes with schema validation. |
| Schemas | TypeBox (the version the pi SDK uses) | One schema library for tool parameters, HTTP bodies and model JSON output. |
| Chat window UI | React 19 in the Electron renderer, bundled by Vite | Each interaction type is a React card in the chat stream. |
| Video engine | Project-Feynman `make-video` (Python 3.14, uv, ManimGX). It runs as an external CLI and is not a dependency. | We already have it. See §4. |

**Project layout.** Organized by feature:

```
pi-sidecar/
  src/
    config.ts             every tunable value and every prompt (single source of truth)
    main/                 Electron main process: window, app lifecycle, IPC handlers
    preload/              contextBridge API: the only bridge between main and the window
    server/               Fastify hook endpoint (runs in the main process)
    builder-events/       parse and filter Claude Code hook payloads
    tutor/                pi agent session, custom tools, intervention decisions
    exercises/            one folder per interaction type: generate + grade
    builder-bridge/       read-only `claude -p` calls to the Builder
    video/                make-video subprocess wrapper and job queue
    learning-state/       session-scoped concept records
  renderer/               React chat window (one card component per interaction type)
  test/                   Vitest suites, fixtures of recorded hook payloads
.claude/settings.json     hook entries that post to Pi (installed into the student's repo)
```

## 3. Deployment model

Pi runs **locally only**. Nothing is hosted.

- `npm start` (Electron Forge) opens the Pi window and starts the hook server on `127.0.0.1:4780`. The student keeps the window beside the terminal running Claude Code. We don't package an installer; the demo runs from source.
- Hosted calls are the Anthropic Messages API (Pi's Haiku calls) and the student's own Claude Code login (Builder questions and video agents).
- **Cloud provider:** none for hosting. Anthropic is the only hosted LLM provider.
- A demo laptop needs Node, Claude Code logged in, an `ANTHROPIC_API_KEY`, and, for videos, the video engine prerequisites in §4.

## 4. Video engine integration

Video generation is an **optional feature, off by default**. The student turns it on with a toggle in the Pi window's settings, and it can be switched off at any time. When it's off, or when the engine isn't available, Pi gives a text or diagram explanation instead. The setting's default lives in `config.ts` (`VIDEO_ENABLED_BY_DEFAULT = false`).

**Videos run through the student's own Claude session.** `make-video` starts headless Claude Code sessions using the Claude account logged in on the laptop, the same login the student uses with Claude Code. Video work therefore counts against the student's Claude plan, not Pi's API key, and Pi's Haiku budget stays small. To guarantee this, Pi removes `ANTHROPIC_API_KEY` from the environment it passes to `make-video`. Otherwise the engine would bill parts of the run to the key. Before turning the toggle on, the window checks that Claude Code is logged in (`claude auth status`) and tells the student that videos use their Claude plan.

When enabled, Pi calls Project-Feynman's CLI as a subprocess:

```sh
uv run make-video "<concept title>" --quality fastest --length 45 \
  --workspace <student repo> --name <job id>
```

- **Speed and usage:** `--quality fastest` takes about 2 to 3 minutes per minute of video. At API prices that would be about $1 to $1.30, and on a subscription it uses a similar share of the student's Claude plan. `--length 45` keeps videos short. `config.ts` caps videos per session (default 3), and the window shows how many are left.
- **Grounding:** `--workspace` lets the engine's facts stage read the student's code in read-only mode, so videos explain the real implementation.
- **Async:** jobs run in a queue (1 at a time). The chat window shows a "video rendering" card and swaps in the MP4 from `out/<name>/` when the process exits 0. The window plays it through a custom `pi-video://` protocol that serves only files from the video output folder.
- **Agent choice:** always pass `--agent claude`, the route that uses the logged-in Claude session. Never use `--agent pi`: it bills to an API key, has no sandbox, and does not accept `--workspace`. Never use `--agent codex`, which needs a different account.
- **Requirements:** macOS with a Metal GPU, uv, Python 3.14, a Project-Feynman checkout at `VIDEO_ENGINE_DIR`, and a logged-in Claude Code. If any are missing, the toggle is greyed out with the reason shown, and Pi uses text explanations.

## 5. Test framework

- **Vitest** for unit and integration tests, and **Testing Library** for React cards.
- **Playwright** (its Electron support, `_electron.launch`) for one end-to-end test of the demo flow: a recorded hook event leads to a quiz card in the window, an answer and feedback.
- **LLM calls are never made in unit tests.** Model access goes through a `TutorModel` interface. Tests use a fake that returns fixed JSON. A separate, opt-in `npm run test:live` suite calls Haiku to check prompt output against schemas.
- **Fixtures:** recorded Claude Code hook payloads in `test/fixtures/hooks/`.
- Run everything with `npm test`. `npm run typecheck` runs `tsc --noEmit`.

## 6. Prohibited libraries

| Library | Reason | Use instead |
|---|---|---|
| LangChain / LangGraph | Adds a second agent framework on top of pi and hides prompts and token use | The pi SDK and `config.ts` prompts |
| `openai` or `@anthropic-ai/sdk` called directly from feature code | Bypasses pi's model runtime and makes the provider hard to swap | pi `ModelRuntime` behind the `TutorModel` interface |
| Express | Weaker typing and no built-in schema validation | Fastify 5 |
| Socket.IO / WebSocket libraries | The window and the main process are in one app, so a socket adds an unneeded network layer | Electron IPC through the preload `contextBridge` API |
| `@electron/remote` | Gives the window direct access to main-process objects, which breaks context isolation | Typed IPC calls exposed by the preload script |
| Next.js or another server-rendered web framework | Pi's window is a local app with no server rendering, and these frameworks fight Electron's file and protocol model | React with Vite |
| axios / node-fetch | Node 24 has `fetch` built in | Global `fetch` |
| Zod | A second schema library next to the one pi uses | TypeBox |
| Prisma / any ORM or database server | State lasts one session. A database adds setup on every laptop. | In-memory state plus a JSONL event log |
| Jest | Slow with ESM and TypeScript, and duplicates Vitest | Vitest |
| lodash / moment | Native JS covers what we need | Native array methods, `Intl`, `Date` |
| `shelljs`, `child_process.exec` with a shell string | Shell injection risk from concept titles and file paths | `child_process.spawn(cmd, argsArray)` with `shell: false` |

## 7. Security basics

- **Secrets:** `ANTHROPIC_API_KEY` and `VIDEO_ENGINE_DIR` live in `.env`, which is gitignored. A committed `.env.example` lists them without values. Keys stay in the Electron main process and never reach the window.
- **Electron hardening:**
  - The window runs with `contextIsolation: true`, `sandbox: true` and `nodeIntegration: false`.
  - The preload script exposes a small typed API (send answer, receive card) and nothing else.
  - The window loads only the app's own bundle, with a strict Content Security Policy.
  - Navigation and new windows are blocked. External links open in the system browser.
- **Local only:** the server binds to `127.0.0.1`, never `0.0.0.0`. The hook endpoint checks a random per-session token that Pi writes into the hook command at startup, so other local pages can't post fake events.
- **Pi never writes to the student's repo:**
  - Pi's agent session gets only the `read`, `grep`, `find` and `ls` tools.
  - A `tool_call` extension handler blocks any path outside the repo and any secret file (`.env*`, `*.pem`, `id_*`, `.git/`).
- **Builder questions are read-only:** `claude -p` runs with `--allowedTools Read,Grep,Glob` and `--permission-mode dontAsk`.
- **Input validation:** every hook payload, IPC message and model JSON output is checked against a TypeBox schema. Bad input is logged and dropped.
- **Subprocesses:** `make-video` and `claude` are started with argument arrays, never a shell string. Concept titles are passed as one argument.
- **Student data:** answers and learning state stay on the laptop in `.pi-sidecar/` (gitignored). Only the code snippets and answers needed for grading are sent to the model.
- **Cost guard:** `config.ts` sets per-session caps on Haiku tokens, Builder calls and videos. Pi stops intervening when a cap is reached. Video is off unless the student turns it on, and its subprocess never receives the API key, so it can only use the student's own Claude login.

## 8. Examples

### Typical endpoint: receive a Claude Code hook event

Claude Code posts its hook payload. Pi filters it, records it and decides whether to intervene.

```typescript
// src/server/builder-events-route.ts
import type { FastifyInstance } from "fastify";
import { Type, type Static } from "typebox";
import { handleBuilderEvent } from "../builder-events/handle-builder-event.js";

const HookPayload = Type.Object({
  session_id: Type.String(),
  hook_event_name: Type.Union([
    Type.Literal("PostToolUse"),
    Type.Literal("Stop"),
    Type.Literal("UserPromptSubmit"),
  ]),
  cwd: Type.String(),
  tool_name: Type.Optional(Type.String()),
  tool_input: Type.Optional(Type.Record(Type.String(), Type.Unknown())),
});
type HookPayload = Static<typeof HookPayload>;

export function registerBuilderEventsRoute(app: FastifyInstance, sessionToken: string) {
  app.post<{ Body: HookPayload }>(
    "/events/claude",
    { schema: { body: HookPayload } },
    async (request, reply) => {
      if (request.headers["x-pi-token"] !== sessionToken) {
        return reply.code(401).send({ error: "invalid token" });
      }
      const decision = await handleBuilderEvent(request.body);
      // Return quickly: Claude Code waits on the hook. Exercise generation runs in the background.
      return reply.code(202).send({ accepted: true, intervention: decision.kind });
    },
  );
}
```

The matching hook entry in the student's `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "curl -s --max-time 2 -X POST http://127.0.0.1:4780/events/claude -H 'content-type: application/json' -H \"x-pi-token: $PI_TOKEN\" --data-binary @-"
          }
        ]
      }
    ]
  }
}
```

### Typical function: generate a grounded multiple-choice question

```typescript
// src/exercises/multiple-choice/generate-multiple-choice.ts
import { Type, type Static } from "typebox";
import { PROMPTS, LIMITS } from "../../config.js";
import type { TutorModel } from "../../tutor/tutor-model.js";
import type { CodeChange } from "../../builder-events/code-change.js";

export const MultipleChoiceQuestion = Type.Object({
  concept: Type.String(),
  question: Type.String(),
  options: Type.Array(Type.String(), { minItems: 4, maxItems: 4 }),
  correctIndex: Type.Integer({ minimum: 0, maximum: 3 }),
  explanation: Type.String(),
  needsBuilderVerification: Type.Boolean(),
});
export type MultipleChoiceQuestion = Static<typeof MultipleChoiceQuestion>;

/**
 * Writes one question about the code that just changed. Haiku drafts it.
 * When the answer depends on runtime behavior Haiku can't confirm from the
 * diff alone, the question is flagged for the Builder to verify first.
 */
export async function generateMultipleChoice(
  model: TutorModel,
  change: CodeChange,
  concept: string,
): Promise<MultipleChoiceQuestion> {
  const snippet = change.diff.slice(0, LIMITS.maxDiffCharacters);
  return model.generateJson(MultipleChoiceQuestion, {
    system: PROMPTS.multipleChoiceSystem,
    user: PROMPTS.multipleChoiceUser({ concept, filePath: change.filePath, snippet }),
    maxTokens: LIMITS.multipleChoiceMaxTokens,
  });
}
```

### Typical test

```typescript
// test/exercises/generate-multiple-choice.test.ts
import { describe, expect, it } from "vitest";
import { generateMultipleChoice } from "../../src/exercises/multiple-choice/generate-multiple-choice.js";
import { FakeTutorModel } from "../fakes/fake-tutor-model.js";
import { loadCodeChange } from "../fixtures/load-code-change.js";

describe("generateMultipleChoice", () => {
  it("sends only the changed snippet, truncated to the configured limit", async () => {
    const model = new FakeTutorModel({
      concept: "connection map",
      question: "What happens when one user opens two tabs?",
      options: ["A", "B", "C", "D"],
      correctIndex: 1,
      explanation: "Each tab opens its own socket.",
      needsBuilderVerification: false,
    });
    const change = loadCodeChange("websocket-connection-map");

    await generateMultipleChoice(model, change, "connection map");

    expect(model.lastRequest.user).toContain(change.filePath);
    expect(model.lastRequest.user).not.toContain("UNCHANGED_FILE_MARKER");
  });

  it("rejects model output that has fewer than four options", async () => {
    const model = new FakeTutorModel({ concept: "x", question: "q", options: ["A"], correctIndex: 0, explanation: "e", needsBuilderVerification: false });
    const change = loadCodeChange("websocket-connection-map");

    await expect(generateMultipleChoice(model, change, "x")).rejects.toThrow(/options/);
  });
});
```

### Typical custom tool: Pi asks the Builder

Pi's agent decides when to escalate. The escalation is a pi custom tool.

```typescript
// src/tutor/ask-builder-tool.ts
import { Type } from "typebox";
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { askBuilder } from "../builder-bridge/ask-builder.js";

export default function registerAskBuilder(pi: ExtensionAPI) {
  pi.registerTool({
    name: "ask_builder",
    label: "Ask the Builder",
    description:
      "Ask Claude Code (read-only) to explain or verify behavior of the student's code. Use only when the answer can't be read directly from the diff.",
    parameters: Type.Object({ question: Type.String(), filePaths: Type.Array(Type.String()) }),
    execute: async (_toolCallId, params, signal) => {
      const answer = await askBuilder(params.question, params.filePaths, signal);
      return { content: [{ type: "text", text: answer }], details: {} };
    },
  });
}
```

## 9. Open technical questions

These follow from the open questions in §13 of the founding idea:

- **Blocking the Builder.** A `UserPromptSubmit` or `PreToolUse` hook can likely hold Claude Code until the student finishes a Pi exercise, by blocking until Pi's endpoint answers. We still need to confirm the hook timeout behavior, and decide when blocking is worth it.
- **Window placement.** Where the Pi window sits next to the terminal, and how it gets the student's attention without breaking their flow. Options include a side-docked window, always-on-top, or a system notification.
- **Video engine access.** `make-video` runs on macOS with a Metal GPU from a Project-Feynman checkout. We need to decide which teammates' laptops run videos, and whether the others share one machine's renders during the demo.
