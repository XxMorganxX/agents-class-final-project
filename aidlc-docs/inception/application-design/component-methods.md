# Component Methods

**Stage:** INCEPTION - Application Design
**Sources:** `components.md`, `application-design-plan.md` (Q4, Q5, Q6, Q9, Q13, Q14 and the answer analysis), requirements.md, stories.md

Method signatures for each component, written in TypeScript. They fix names, inputs, outputs and failure shapes so units can be built in parallel. **Business rules are not decided here.** Where a method needs a rule (label thresholds, grading verdicts, recovery steps, cadence, how the installer restores the backup), the rule is left to Functional Design for the unit that owns it. Those places are marked *FD*.

Conventions:
- Methods that can fail in an expected way return `Result<T, E>` (`{ ok: true; value: T } | { ok: false; error: E }`) instead of throwing. Only programmer errors throw.
- Every input that crosses a boundary (HTTP, IPC, model output, files) is checked against a C2 schema first.
- `signal: AbortSignal` is passed to anything that can time out.
- Pure functions are marked **pure**: no I/O, no clock, no randomness. These are the property-based test targets (NFR-8.3).

## Shared types (C2)

```ts
type CardId = string;            // unique per session
type ConceptId = string;         // must be in CONFIG.concepts
type ComponentType = string;     // registered in C3
type Iso = string;               // ISO 8601 timestamp

interface CodeRef { filePath: string; startLine: number; endLine: number }
interface CodeSnapshot extends CodeRef { snippet: string; fileHash: string; takenAt: Iso }

type Grade =
  | { kind: "right-wrong"; correct: boolean; correctAnswer: string; explanation: string }
  | { kind: "checklist"; verdict: "got-it" | "partly" | "not-yet"; hit: string[]; missed: string[] }
  | { kind: "prediction"; outcome: "right" | "wrong" | "no-result" }
  | { kind: "skipped"; reason: "student" | "timeout" };

type MasteryLabel = "not-yet-shown" | "shaky" | "solid";
type CapKind = "haikuUsd" | "builderCalls" | "videos";

// Builder events (C8)
type BuilderEvent =
  | { kind: "file-changed"; filePath: string; toolName: "Edit" | "Write"; at: Iso }
  | { kind: "turn-ended"; endMessage: string; testRuns: TestRun[]; at: Iso }
  | { kind: "prompt-sent"; prompt: string; at: Iso };
interface TestRun { command: string; exitCode: number; output: string }

// Triggers (FR-0.2)
type Trigger =
  | { kind: "builder-activity"; fileChanges: string[]; prompts: string[]; turnEnded: boolean; testRuns: TestRun[] }
  | { kind: "student-response"; cardId: CardId; grade: Grade; nextStep: "partly-follow-up" | "drill-down-question" | "retest" }
  | { kind: "end-session"; conceptsCovered: ConceptId[] };
```

## C1 Config

```ts
const CONFIG: Readonly<PiConfig>;
function validateConfig(config: unknown): Result<PiConfig, { path: string; message: string }[]>;
```
`PiConfig` groups `caps`, `timeouts`, `agent` (step limit, run timeout), `cadence` (exercises per turn, countdown seconds), `latency` (15 s, 60 s), `concepts`, `prompts`, `hookServer` (port), `video` (enabled by default, max length).

## C2 Shared Schemas

```ts
function check<S extends TSchema>(schema: S, value: unknown): Result<Static<S>, SchemaError[]>;
const HookPayload: TSchema;      // PostToolUse | Stop | UserPromptSubmit
const RuntimeFile: TSchema;      // { port, token, pid }
const IpcRequest: TSchema;       // union of window -> main messages
const IpcEvent: TSchema;         // union of main -> window events
```

## C3 Component Registry (contract)

```ts
interface ComponentTypeDefinition<F, R> {
  type: ComponentType;
  fields: TSchema;                 // what the agent fills in
  response: TSchema | null;        // null for display-only types
  isExercise: boolean;             // counts toward the per-turn limit
  refersToCode: boolean;           // C10 takes a CodeSnapshot
  answerKey?(fields: F): { question: string; key: string };  // for Builder checks and receipt hashes
}

interface MainComponentType<F, R> extends ComponentTypeDefinition<F, R> {
  grade?(input: GradeInput<F, R>): Promise<Result<Grade, GradeError>>;
}
interface GradeInput<F, R> { fields: F; response: R; snapshot?: CodeSnapshot; model: TutorModel; signal: AbortSignal }

class ComponentRegistry {                         // main
  register(def: MainComponentType<any, any>): void;     // throws on duplicate type
  unregister(type: ComponentType): void;                // used by C21 deactivate
  get(type: ComponentType): MainComponentType<any, any> | undefined;
  list(): ComponentType[];
  validateFields(type: ComponentType, fields: unknown): Result<unknown, SchemaError[]>;
  validateResponse(type: ComponentType, response: unknown): Result<unknown, SchemaError[]>;
  showComponentSchema(): TSchema;                       // union of all registered types (Q4 A)
}

interface CardProps<F, R> {
  card: CardView<F>;
  respond(response: R): void;
  act(action: CardAction): void;
}
class CardRegistry {                              // renderer
  register(type: ComponentType, Card: React.ComponentType<CardProps<any, any>>): void;
  get(type: ComponentType): React.ComponentType<CardProps<any, any>>;  // unknown type -> plain-text fallback card
}
```

## C4 Lesson Component Types

Each folder exports a `MainComponentType` from `schema.ts` plus `grader.ts`, and a card from `Card.tsx`.

| Type | `fields` (summary) | `response` | `grade` |
|---|---|---|---|
| explanation | `text`, `codeRef`, `terms[]` | none | none |
| multiple-choice | `question`, `options[4]`, `correctIndex`, `explanation`, `needsBuilderVerification`, `verificationReceiptId?`, `codeRef`, `architectureCompare?` | `{ choice: number }` | right/wrong *FD* |
| written-response | `prompt`, `checklist[]`, `codeRef` | `{ text: string }` | checklist through `model.generateJson` *FD* |
| quick-time | as multiple-choice, plus `countdownSeconds` (from config) | `{ choice: number } \| { timedOut: true }` | right/wrong; timeout gives `skipped` |
| system-design-graph | `title`, `mermaid` | none | none |
| data-graphic | `kind: "bar" \| "line" \| "table"`, `data` (inline rows) | none | none |
| prediction | `phase: "before" \| "after"`, `situation`, `options?`, `codeRef` | `{ choice: number } \| { text: string }` | `judgePrediction` when the actual result arrives *FD* |
| session-summary | `SessionSummary` (built by C15, never by the model) | none | none |

```ts
// prediction/grader.ts
function judgePrediction(input: { fields: PredictionFields; prediction: PredictionResponse; actual: ActualResult; model: TutorModel; signal: AbortSignal }): Promise<Result<Grade, GradeError>>;   // FD: when a model judge is needed
interface ActualResult { text: string; source: "student" | "builder-test-run" }
```

## C5 Hook Server

```ts
class HookServer {
  constructor(deps: { config: PiConfig; token: string; onPayload(p: HookPayload): void; log: EventLog });
  start(): Promise<Result<{ port: number }, "port-in-use" | "bind-failed">>;   // binds 127.0.0.1 only
  stop(): Promise<void>;
}
// Route: POST /events/claude -> 401 wrong token, 400 bad schema, 202 accepted (before any teaching work)
```

## C6 Hook Installer

```ts
interface HookEntryPreview { event: "PostToolUse" | "Stop" | "UserPromptSubmit"; matcher?: string; command: string }
type InstallerError = "settings-not-json" | "write-failed" | "cancelled";

class HookInstaller {
  constructor(deps: { settingsPath: string; appDataDir: string; fs: FileSystem; log: EventLog });
  preview(): Promise<Result<HookEntryPreview[], InstallerError>>;
  install(confirmed: true): Promise<Result<{ backupPath: string }, InstallerError>>;
  remove(): Promise<Result<{ removed: number; restoredFromBackup: boolean }, InstallerError>>;   // removed = 0 -> "no Pi hooks found"
  status(): Promise<"installed" | "not-installed" | "partial">;
  writeRuntimeFile(): Promise<{ token: string; port: number }>;    // new random token each launch, mode 0600
  deleteRuntimeFile(): Promise<void>;
}

// pure (property-tested: merge then remove round-trips; other keys untouched)
function mergePiHooks(settings: JsonObject, entries: HookEntryPreview[]): JsonObject;
function removePiHooks(settings: JsonObject): { settings: JsonObject; removed: number };
function isPiEntry(entry: unknown): boolean;
// FD: when remove() restores the byte-exact backup vs removes entries structurally
```

```ts
// resources/pi-hook.mjs (runs under `node`, no Pi imports)
// main(eventName): read stdin, read runtime file, POST with x-pi-token, abort under 200 ms,
// swallow every error, write nothing, exit 0.
```

## C7 Setup Checks

```ts
type SetupStep = "repo" | "hooks" | "claude-code" | "api-key";
interface StepResult { step: SetupStep; status: "pass" | "fail" | "not-run"; missing?: string; fixHint?: string }   // never contains the key value

function readEnv(path: string): Result<{ anthropicApiKey: string; videoEngineDir?: string }, { missing: "ANTHROPIC_API_KEY" }>;

class SetupChecks {
  constructor(deps: { env: EnvValues; installer: HookInstaller; model: TutorModel; spawn: SpawnFn; store: AppDataStore });
  runStep(step: SetupStep): Promise<StepResult>;
  runAll(): Promise<StepResult[]>;          // in order: repo, hooks, claude-code, api-key
  chooseRepo(path: string): Promise<StepResult>;
  currentRepo(): string | undefined;
}
```

## C8 Builder Events (pure)

```ts
function toBuilderEvent(payload: HookPayload, repoRoot: string): BuilderEvent | { ignored: "outside-repo" | "irrelevant-tool" };
function mergeActivity(events: BuilderEvent[]): Extract<Trigger, { kind: "builder-activity" }>;
function readTurnDetails(transcriptPath: string, fs: FileSystem): Promise<{ endMessage: string; testRuns: TestRun[] }>;   // not pure; reads only the transcript Claude Code already wrote
```

## C9 Session Orchestrator (service)

```ts
type CardAction = "skip" | "dispute" | "show-answer" | "drill-down" | "accept-video" | "close";

class SessionOrchestrator {
  constructor(deps: {
    agent: TutorAgent; pipeline: CardPipeline; record: LearningRecord; budget: BudgetLedger;
    registry: ComponentRegistry; feed: FeedSink; log: EventLog; clock: Clock; config: PiConfig;
  });
  handleBuilderEvent(event: BuilderEvent): void;                         // queues or merges (Q8 A)
  handleResponse(cardId: CardId, response: unknown): Promise<Result<Grade, ResponseError>>;   // only the first response is graded (US-18)
  handleCardAction(cardId: CardId, action: CardAction): Promise<Result<void, ActionError>>;
  addActualResult(cardId: CardId, result: ActualResult): Promise<Result<Grade, ResponseError>>;
  endSession(): Promise<void>;                                            // after cards, then summary
  snapshot(): FeedSnapshot;                                               // for getSnapshot() (Q10 C)
}

interface FeedSink { cardAdded(card: CardView<unknown>): void; cardUpdated(patch: CardPatch): void; statusChanged(status: StatusView): void }

// session/cadence.ts (pure)  FD: exact rules
function mayShow(type: ComponentTypeDefinition<any, any>, turn: TurnState): { allowed: true } | { allowed: false; reason: "exercise-before-stop" | "turn-limit" };
// session/recovery.ts (pure)  FD: exact steps
function nextRecoveryStep(state: RecoveryState, event: RecoveryEvent): { state: RecoveryState; step: "reveal-answer" | "agent-follow-up" | "agent-drill-question" | "agent-retest" | "done" };
// session/run-queue.ts
class RunQueue { enqueue(trigger: Trigger): void; onIdle(): Promise<void> }   // one run at a time; merges builder-activity only
```

## C10 Card Pipeline (service)

```ts
interface ProposedComponent {
  type: ComponentType; concept: ConceptId; fields: unknown;
  codeRef?: CodeRef; needsBuilderVerification: boolean; verificationReceiptId?: string;
}
type DropReason = "invalid-twice" | "builder-disagrees" | "builder-timeout" | "builder-cap" | "latency-limit" | "unknown-concept" | "cadence";

class CardPipeline {
  constructor(deps: { registry: ComponentRegistry; builder: BuilderBridge; repo: RepoReader; clock: Clock; log: EventLog; config: PiConfig });
  propose(p: ProposedComponent, ctx: { triggerAt: Iso; regenerate(errors: SchemaError[]): Promise<ProposedComponent | null>; signal: AbortSignal }):
    Promise<Result<CardRecord, DropReason>>;
  markCodeChanged(filePath: string): CardPatch[];
  hasCodeChanged(card: CardRecord): Promise<boolean>;      // compares stored fileHash with a fresh read
}

interface CardRecord {
  cardId: CardId; type: ComponentType; concept: ConceptId; fields: unknown;
  snapshot?: CodeSnapshot; receipt?: VerificationReceipt; codeChanged: boolean; shownAt: Iso;
}

// pure
function receiptCovers(receipt: VerificationReceipt, answerKey: { question: string; key: string }): boolean;  // verdict agrees and hash matches (analysis 5)
function hashAnswerKey(answerKey: { question: string; key: string }): string;
```

## C11 Tutor Agent

```ts
interface AgentRunContext {
  trigger: Trigger;
  recordSummary: RecordSummaryForAgent;    // per-concept labels, next-lesson hints, questions already asked or skipped
  turn: TurnState;                         // so the agent knows if exercises are allowed
  videosAvailable: boolean;
}
type RunOutcome =
  | { kind: "shown"; card: CardRecord }
  | { kind: "nothing-to-teach"; reason: string }
  | { kind: "step-limit" } | { kind: "cap-reached"; cap: CapKind } | { kind: "dropped"; reason: DropReason }
  | { kind: "failed"; error: ModelError };

class TutorAgent {
  constructor(deps: { model: TutorModel; tools: AgentToolSet; pipeline: CardPipeline; log: EventLog; config: PiConfig });
  run(ctx: AgentRunContext, signal: AbortSignal): Promise<RunOutcome>;
}

interface AgentToolSet { list(): AgentTool[]; add(tool: AgentTool): void; remove(name: string): void }   // C21 adds offer_video
// show_component: parameters = registry.showComponentSchema(); execute -> pipeline.propose(); a shown card ends the run
// concept tags not in CONFIG.concepts are dropped before propose()
```

## C12 TutorModel (contract)

```ts
interface TokenUsage { inputTokens: number; outputTokens: number; usd: number }
type ModelError = { kind: "network" | "rate-limit" | "server" | "timeout" | "bad-output" | "cap-reached"; retriable: boolean; message: string };

interface TutorModel {
  generateJson<S extends TSchema>(schema: S, request: { system: string; user: string; maxTokens: number }, signal: AbortSignal):
    Promise<Result<{ value: Static<S>; usage: TokenUsage }, ModelError>>;
  runAgent(request: {
    system: string; message: string; tools: AgentTool[]; maxSteps: number; timeoutMs: number;
  }, signal: AbortSignal): Promise<Result<{ stopReason: "tool-ended" | "no-action" | "step-limit"; toolCalls: ToolCallLog[]; usage: TokenUsage }, ModelError>>;
}

class PiTutorModel implements TutorModel { constructor(deps: { apiKey: string; modelId: "claude-haiku-5-5"; retry: RetryPolicy }) }
class BudgetedTutorModel implements TutorModel { constructor(inner: TutorModel, budget: BudgetLedger) }   // reserve, call, settle actual usd
class FakeTutorModel implements TutorModel { constructor(script: FakeScript); readonly requests: unknown[] }   // test/fakes
```

## C13 Repo Read Tools

```ts
type GuardResult = { allowed: true; absolutePath: string } | { allowed: false; reason: "outside-repo" | "secret-file" | "not-found" };

function guardPath(repoRoot: string, requested: string, realpath: (p: string) => string): GuardResult;   // pure given realpath; property-tested
function createRepoTools(repoRoot: string, deps: { fs: FileSystem; log: EventLog }): AgentTool[];        // read, grep, find, ls

interface RepoReader {                                // used by C10
  readRange(ref: CodeRef): Promise<Result<CodeSnapshot, GuardResult>>;
  hashFile(filePath: string): Promise<Result<string, GuardResult>>;
}
```

## C14 Builder Bridge (contract)

```ts
interface VerificationReceipt { receiptId: string; verdict: "agrees" | "disagrees"; answerKeyHash: string; builderAnswer: string; at: Iso }
type BridgeError = "timeout" | "cap-reached" | "cli-missing" | "not-logged-in" | "failed";

interface BuilderBridge {
  checkAnswerKey(req: { question: string; key: string; filePaths: string[] }, signal: AbortSignal): Promise<Result<VerificationReceipt, BridgeError>>;
  getReceipt(receiptId: string): VerificationReceipt | undefined;
  isAvailable(): Promise<boolean>;
}
// ask_builder tool: parameters { question, proposedKey, filePaths } -> checkAnswerKey -> returns verdict, builderAnswer, receiptId to the agent

class ClaudeCliBuilderBridge implements BuilderBridge { constructor(deps: { repoRoot: string; spawn: SpawnFn; budget: BudgetLedger; config: PiConfig; log: EventLog }) }
class FakeBuilderBridge implements BuilderBridge {}
// argv: ["-p", prompt, "--allowedTools", "Read,Grep,Glob", "--permission-mode", "dontAsk", "--output-format", "json"], cwd = repoRoot, shell: false
```

## C15 Learning Record

```ts
type LearningEvent =
  | { kind: "question-asked"; cardId: CardId; concept: ConceptId; type: ComponentType; at: Iso }
  | { kind: "explained"; cardId: CardId; concept: ConceptId; at: Iso }
  | { kind: "answered"; cardId: CardId; at: Iso }                       // answer text only goes to the debug log
  | { kind: "graded"; cardId: CardId; grade: Grade; at: Iso }
  | { kind: "skipped"; cardId: CardId; reason: "student" | "timeout"; at: Iso }
  | { kind: "disputed"; cardId: CardId; at: Iso }
  | { kind: "prediction-made"; cardId: CardId; concept: ConceptId; phase: "before" | "after"; at: Iso }
  | { kind: "actual-result-added"; cardId: CardId; source: "student" | "builder-test-run"; at: Iso }
  | { kind: "card-closed"; cardId: CardId; openMs: number; at: Iso }
  | { kind: "turn-ended"; at: Iso };

class LearningRecord {
  constructor(deps: { log: EventLog });
  append(event: LearningEvent): void;          // only the orchestrator calls this
  events(): readonly LearningEvent[];
}

// learning-record/rules.ts (all pure; property-tested)   FD: exact rules
function masteryLabel(events: readonly LearningEvent[], concept: ConceptId): MasteryLabel;
function nextLessonHint(events: readonly LearningEvent[], concept: ConceptId): "diagnose-first" | "explain-first" | "practice";
function wasAskedOrSkipped(events: readonly LearningEvent[], questionKey: string): boolean;
function conceptsCovered(events: readonly LearningEvent[]): ConceptId[];
function predictionPairs(events: readonly LearningEvent[]): PredictionPair[];
function sessionMetrics(events: readonly LearningEvent[], budget: BudgetSnapshot): SessionMetrics;
function buildSummary(events: readonly LearningEvent[], budget: BudgetSnapshot): SessionSummary;   // becomes a session-summary card's fields
function summarizeForAgent(events: readonly LearningEvent[]): RecordSummaryForAgent;

interface PredictionPair { concept: ConceptId; before: "right" | "wrong" | "no-result" | "skipped" | "not-asked"; after: "right" | "wrong" | "no-result" | "skipped" | "not-asked" }
interface SessionMetrics {
  prediction: { beforeRight: number; afterRight: number; pairs: number };
  comprehension: { teachBacks: number; gotItOrPartly: number };
  interruption: { medianOpenMs: number; skipped: number; cardsPerTurn: number };
  efficiency: { haikuUsd: number; builderCalls: number; videos: number; predictionGain: number };
}
```

## C16 Budget Ledger

```ts
interface Reservation { id: string; cap: CapKind; amount: number }
interface BudgetSnapshot { used: Record<CapKind, number>; limits: Record<CapKind, number>; reached: CapKind[] }

class BudgetLedger {
  constructor(limits: Record<CapKind, number>);
  tryReserve(cap: CapKind, amount: number): Result<Reservation, { capReached: CapKind }>;
  settle(reservation: Reservation, actual: number): void;     // a settled or released reservation is ignored after (never counted twice)
  release(reservation: Reservation): void;
  snapshot(): BudgetSnapshot;
  onCapReached(listener: (cap: CapKind) => void): () => void;  // fires once per cap
}
// FD: how a Haiku reservation is estimated before a call
```

## C17 Event Log

```ts
interface AgentRunLog { runId: string; trigger: Trigger["kind"]; toolCalls: { name: string; ms: number; ok: boolean }[]; outcome: RunOutcome["kind"]; reason?: string; ms: number; usage: TokenUsage }

class EventLog {
  constructor(deps: { dir: string; debug: () => boolean; fs: FileSystem; clock: Clock });
  logRun(run: AgentRunLog): void;
  logEvent(kind: string, meta: Record<string, string | number | boolean>): void;   // metadata only
  logContent(kind: "prompt" | "model-output" | "student-answer", content: string): void;   // no-op unless debug
  onWriteFailure(listener: (error: Error) => void): () => void;  // fires once
}
```

## C18 Window Shell and IPC Gateway

```ts
function createMainWindow(deps: { preloadPath: string; config: PiConfig }): BrowserWindow;   // hardened settings, CSP, navigation blocked

class IpcGateway implements FeedSink {
  constructor(deps: { window: BrowserWindow; orchestrator: SessionOrchestrator; setup: SetupChecks; installer: HookInstaller; settings: SettingsStore; video?: VideoPlugin; log: EventLog });
  register(): void;                // ipcMain.handle for every request channel; each request checked against IpcRequest
  cardAdded(card: CardView<unknown>): void;
  cardUpdated(patch: CardPatch): void;
  statusChanged(status: StatusView): void;
  setupChanged(results: StepResult[]): void;
}
```

## C19 Preload API

```ts
interface PiApi {
  // window -> main
  getSnapshot(): Promise<FeedSnapshot>;
  respond(cardId: CardId, response: unknown): Promise<Result<Grade, ResponseError>>;
  act(cardId: CardId, action: CardAction): Promise<Result<void, ActionError>>;
  addActualResult(cardId: CardId, result: { text: string }): Promise<Result<Grade, ResponseError>>;
  endSession(): Promise<void>;
  setup: { runAll(): Promise<StepResult[]>; runStep(step: SetupStep): Promise<StepResult>; chooseRepo(): Promise<StepResult> };
  hooks: { preview(): Promise<Result<HookEntryPreview[], InstallerError>>; install(): Promise<Result<void, InstallerError>>; remove(): Promise<Result<{ removed: number }, InstallerError>> };
  settings: { get(): Promise<SettingsView>; setDebug(on: boolean): Promise<void>; setVideos(on: boolean): Promise<VideoCheckView> };
  // main -> window
  onCardAdded(cb: (card: CardView<unknown>) => void): () => void;
  onCardUpdated(cb: (patch: CardPatch) => void): () => void;
  onStatus(cb: (status: StatusView) => void): () => void;
  onSetup(cb: (results: StepResult[]) => void): () => void;
}
// exposed as window.pi; nothing else is exposed
```

## C20 Renderer App

```tsx
function App(): JSX.Element;                         // routes between SetupScreen and Feed
function SetupScreen(props: { results: StepResult[] }): JSX.Element;
function Feed(props: { cards: CardView<unknown>[]; registry: CardRegistry }): JSX.Element;
function StatusBar(props: { status: StatusView }): JSX.Element;      // preparing, model notice, cap notice, videos left
function SettingsPanel(props: { settings: SettingsView }): JSX.Element;
function useFeed(api: PiApi): { cards: CardView<unknown>[]; status: StatusView };   // getSnapshot on mount, then applies events
```

## C21 Video Plug-in (optional; Video Runner contract)

```ts
interface RequirementCheck { item: "macos-metal" | "uv" | "python-3.14" | "video-engine-dir" | "claude-login"; ok: boolean; fix?: string }
type VideoJobResult = { ok: true; mp4Path: string } | { ok: false; reason: "render-failed" | "timeout" | "cancelled" };

interface VideoRunner {                                        // contract (NFR-7.1)
  checkRequirements(): Promise<RequirementCheck[]>;
  enqueue(job: { jobId: string; conceptTitle: string; repoRoot: string }): void;   // one at a time
  onFinished(cb: (jobId: string, result: VideoJobResult) => void): () => void;
  cancelAll(): Promise<void>;
}
class MakeVideoRunner implements VideoRunner {}   // argv: ["run", "make-video", title, "--quality", "fastest", "--length", "45", "--workspace", repo, "--name", jobId, "--agent", "claude"], env without ANTHROPIC_API_KEY, shell: false
class FakeVideoRunner implements VideoRunner {}

interface PluginContext { registry: ComponentRegistry; tools: AgentToolSet; budget: BudgetLedger; orchestrator: SessionOrchestrator; log: EventLog; protocol: ProtocolRegistrar }
// Renderer side: the video Card files are registered in CardRegistry at startup but only ever receive cards
// once main has registered the video types, so the renderer needs no plug-in switch of its own.
class VideoPlugin {
  constructor(runner: VideoRunner, config: PiConfig);
  enable(ctx: PluginContext): Promise<Result<void, RequirementCheck[]>>;   // checks first; registers only on success
  disable(): Promise<void>;                                                  // unregisters tool, types, protocol
  videosLeft(): number;
}
```

## C22 Composition Root

```ts
interface AppOverrides { model?: TutorModel; builder?: BuilderBridge; videoRunner?: VideoRunner; clock?: Clock; fs?: FileSystem; spawn?: SpawnFn; appDataDir?: string; settingsPath?: string }
function createApp(overrides?: AppOverrides): App;
interface App { start(): Promise<Result<void, StartupError>>; stop(): Promise<void>; orchestrator: SessionOrchestrator }   // tests drive the orchestrator directly
type StartupError = { kind: "config-invalid"; problems: string[] } | { kind: "missing-env"; variable: string } | { kind: "port-in-use" };
```

## Left to Functional Design

| Rule | Owning component |
|---|---|
| Mastery label thresholds beyond "solid needs two exercise types" | C15 |
| Next-lesson hint and "already asked" rules | C15 |
| Prediction judging (when a model judge is needed for free-text predictions) | C4 prediction |
| Checklist grading prompt and verdict thresholds | C4 written-response |
| Cadence rules, including what counts as a "significant file" | C9 |
| Recovery steps after "partly" and "not yet" | C9 |
| Haiku reservation estimate before a call | C16, C12 |
| When removal restores the byte-exact backup | C6 |
| Exact hook entry command text, including the missing-script case | C6 |
| Metric formulas (median vs mean open time, prediction gain) | C15 |
