---
name: tauri-rust-react
description: Build Tauri v2 desktop apps with Rust backend and React/TypeScript frontend. Covers architecture, typed IPC, state machines, security, and conventions. Use when asked to "build desktop app", "create Tauri project", "add Tauri command", "add IPC command", "create Rust service", or "add React feature".
user-invocable: /tauri
model: inherit
---

# Tauri + Rust + React Skill

## When to Use This Skill

Use this skill when building or evolving a Tauri v2 desktop app with a Rust backend and a React + TypeScript frontend. It captures the stack requirements, architecture patterns, IPC/type-sharing approach, and conventions for reliable cross-platform apps.

> **IMPORTANT**
>
> - **Commands are thin wrappers**: Zero business logic — all domain logic lives in `services/`.
> - **All IPC validated**: Every `invoke()` response passes through a Zod schema via `invokeCommand`.
> - **No unwrap in commands/services**: Use `?` or explicit error handling.
> - **No unsafe-eval in CSP**: Enables arbitrary code execution in the webview.
> - **All frontend data is untrusted**: Sanitize before use in shell commands, file paths, SQL, and HTML.
> - **Read before write**: Read 2+ existing files of the same type before creating a new one.

## Stack Requirements

- Tauri v2 app shell with a Rust backend for system-level operations and IPC.
- Bun as the JavaScript package manager/dev server for the React frontend.
- React 19+ with functional components and hooks only.
- TypeScript in strict mode for all frontend code.
- Tailwind CSS for styling.
- shadcn/ui for base components (accessible, customizable).
- React Router with `HashRouter` for page navigation.
- TanStack Query for server state (data from Rust backend).
- Zustand for client state (UI state, preferences, ephemeral selections).
- Zod for runtime schema validation at IPC and form boundaries.

## Examples

### Example: Add a New Tauri Command

**user:** "Add a command to list all projects"

**assistant:** "Reading existing commands in `src-tauri/src/commands/` to match patterns...

Found `workspaces.rs` as reference. I'll create `projects.rs` following the same structure:
1. Thin command handler calling `project_service.list()`
2. Plugin registration via `init()` returning `TauriPlugin`
3. Frontend API wrapper in `features/projects/api/index.ts` with Zod validation
4. TanStack Query hook in `features/projects/hooks/useProjects.ts`"

### Example: Add a State Machine

**user:** "The download feature needs states: idle, downloading, paused, complete, failed"

**assistant:** "I'll model this as a tagged enum in Rust with a matching TypeScript discriminated union:
1. `DownloadState` enum in `src-tauri/src/models/download.rs` with `#[serde(tag = \"state\")]`
2. Matching TS type in `features/downloads/types.ts`
3. Transition validation in `download_service.rs` (not the command handler)
4. State change events emitted via `app.emit()` for frontend reactivity"

### Example: Fix Security Issue

**user:** "The open-folder command takes a path from the frontend"

**assistant:** "Frontend paths are untrusted. I'll add input validation:
1. Canonicalize with `std::fs::canonicalize()`
2. Verify the resolved path starts with an allowed root directory
3. Use `Command::new(binary).arg(path)` — never interpolate into shell strings
4. Return `AppError::Validation` if path is outside allowed bounds"

## Protocol

### Step 1: Discover

Read at least 2 existing files of the same type (command, service, component, hook) before creating a new one. Match import order and grouping, error handling pattern, return type wrapping, and naming conventions.

### Step 2: Plan

Identify the target layer (command/service/repository/model/platform for Rust; feature/shared for React). Determine which existing patterns apply. Output an understanding report before writing code.

### Step 3: Implement

Follow the architecture and conventions defined in this skill. Commands are thin wrappers. Services hold logic. Frontend components never call `invoke()` directly. All IPC goes through `invokeCommand` with Zod validation.

### Step 4: Verify

- Rust: `cargo clippy` passes, no `unwrap()`/`expect()` in commands or services
- Frontend: `bun run lint` passes, no cross-feature imports except `import type`
- IPC: every command response validated by a Zod schema
- State machines: transition logic lives in services, not commands

## Rust Backend

### Architecture & Directory Layout

Use a layered, module-driven architecture that keeps business logic out of the Tauri commands.

```
src-tauri/src/
├── main.rs          # App entry point — wiring only
├── state.rs         # AppState struct (shared dependencies)
├── errors.rs        # Unified AppError enum
├── commands/        # Thin invoke handlers (no business logic)
├── services/        # Domain/business logic
├── repositories/    # Persistence and data access
├── models/          # Domain structs and methods
└── platform/        # OS-specific implementations
```

- **Commands**: validate input, call services, return serialized results. No domain decisions, state transitions, or orchestration.
- **Services**: domain logic, process management, coordination. Can depend on repositories and other services.
- **Repositories**: persistence access with traits to allow swapping implementations for tests.
- **Models**: data structs plus domain-specific methods and invariants.
- **Platform**: OS-specific logic behind traits so the rest of the app is platform-agnostic.
- Wire dependencies in `main.rs` and share via Tauri `State`. Keep `main.rs` as wiring only.
- Group services into a `ServiceRegistry`. Group by domain (e.g., `InfraServices { caddy, dns, tls }`) rather than listing flat fields.

### Command Registration

Use Tauri's internal plugin system so each feature module registers its own handlers:

```rust
// commands/workspaces.rs — each module exports a plugin
pub fn init() -> TauriPlugin<tauri::Wry> {
    Builder::new("workspaces")
        .invoke_handler(generate_handler![list_workspaces, create_workspace])
        .build()
}

// main.rs — register plugins, not individual handlers
tauri::Builder::default()
    .plugin(commands::workspaces::init())
    .plugin(commands::projects::init())
    .run(tauri::generate_context!())
    .expect("error running app");
```

When using plugins, frontend invoke calls use the plugin prefix: `invoke("plugin:workspaces|list_workspaces")`.

### Async vs Sync Commands

| Type            | Signature  | Use When                                                           |
| --------------- | ---------- | ------------------------------------------------------------------ |
| Async (default) | `async fn` | Network I/O, file I/O, process spawning, anything that might block |
| Sync            | `fn`       | Pure computation or in-memory lookups completing in microseconds   |

- Use `tauri::async_runtime::spawn()` instead of `tokio::spawn()` — Tauri manages its own runtime.
- Use `tokio::task::spawn_blocking` for CPU-bound blocking work (offloading work that would block the async runtime).

### Error Handling

- Use a single `AppError` enum (with `thiserror` + `Serialize`) for all backend errors.
- Serialize with `#[serde(tag = "kind", content = "detail")]` so the frontend can match on error kind.
- Categorize variants: `Validation` (user-fixable), `NotFound`, `System` (infrastructure), `Transient` (retryable).
- Each variant should carry enough context for the frontend to display a meaningful message (e.g., `NotFound { entity: "workspace", id: String }`).
- Never use `unwrap()` or `expect()` in `commands/` or `services/` modules — use `?` or explicit handling.

### State Machines

Model process lifecycle and multi-step workflows as state machines. Use internally-tagged Rust enums — write the corresponding TypeScript discriminated unions manually.

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(tag = "state")]
pub enum ProcessState {
    Idle,
    Starting,
    Running { pid: u32, started_at: i64 },
    Stopping,
    Stopped { exit_code: i32 },
    Errored { error: String, retry_count: u32 },
}
```

Corresponding TypeScript discriminated union (written manually):

```ts
export type ProcessState =
  | { state: "Idle" }
  | { state: "Starting" }
  | { state: "Running"; pid: number; started_at: number }
  | { state: "Stopping" }
  | { state: "Stopped"; exit_code: number }
  | { state: "Errored"; error: string; retry_count: number };
```

Validate transitions in service methods, not command handlers:

```rust
impl ProcessService {
    pub async fn start(&self, name: &str) -> Result<ProcessState, AppError> {
        let mut current = self.state.lock().await;
        match *current {
            ProcessState::Idle | ProcessState::Stopped { .. } | ProcessState::Errored { .. } => {
                *current = ProcessState::Starting;
                Ok(current.clone())
            }
            ref state => Err(AppError::InvalidTransition {
                from: state.display_name().into(),
                action: "start".into(),
            }),
        }
    }
}
```

- Add a `display_name()` method on state enums for user-friendly error messages instead of `{:?}` Debug format.
- Add an `InvalidTransition { from: String, action: String }` variant to `AppError` for state machine violations.
- Emit state change events via `app.emit("service:state-changed", &new_state)` from command handlers.

### IPC Channels for Streaming

For long-running operations (file exports, installations, scans), use Tauri v2 channels instead of repeated events. Channel payloads are NOT events — events are for `app.emit()`/`listen()`; channels are for streaming command responses.

```rust
use tauri::ipc::Channel;

#[tauri::command]
async fn export_workspace(
    workspace_id: String,
    on_progress: Channel<ExportProgress>,
) -> Result<String, AppError> {
    for i in 0..total {
        on_progress.send(ExportProgress { current: i, total, percent: (i * 100) / total })?;
    }
    Ok("export_path.zip".into())
}
```

- Prefer channels over events for streaming/progress data.
- Use events for state change notifications (service started, file changed).

### Security

#### CSP Configuration

Configure Content Security Policy in `tauri.conf.json`. Use restrictive defaults:

```json
{
  "app": {
    "security": {
      "csp": "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' asset: https:; connect-src ipc: http://ipc.localhost"
    }
  }
}
```

- Never use `unsafe-eval` in production — it enables arbitrary code execution.
- Do not use `'unsafe-inline'` in `style-src`. Use nonce-based CSP (`'nonce-<random>'`) instead.
- Restrict `connect-src` to only the origins your app actually communicates with. Add specific domains only when needed.

#### Capabilities Model

- Tauri v2 uses fine-grained, per-window permissions via capability files in `src-tauri/capabilities/`.
- Each capability file defines `$schema`, `identifier`, `description`, `windows` (target window names), and `permissions` (e.g., `core:default`, `fs:allow-read-file`).
- **Default-deny**: start with no permissions, add explicitly as features require them.
- Each window gets only the permissions it needs — secondary windows should have a separate, more restrictive capability file.
- Scope filesystem access to specific directories using permission scoping (e.g., `$APP_DATA_DIR`).

#### Secret Storage

- Store all secrets (tokens, keys, credentials) in the OS keychain via a Tauri secure storage plugin.
- Never store secrets in config files, localStorage, or application state that gets serialized.
- Enforce allowed directory roots — never allow path traversal beyond the app data directory or user-selected paths.
- Validate all IPC inputs in command handlers before passing to services.

#### IPC Input Sanitization

> **IMPORTANT**
>
> - **Shell commands**: never interpolate user input into command strings. Use `Command::new(binary).arg(user_input)` for each argument separately.
> - **File paths**: canonicalize with `std::fs::canonicalize()`, then verify the resolved path starts with an allowed root directory.
> - **SQL**: use parameterized queries exclusively. Never build SQL with `format!()` or string concatenation.
> - **HTML output**: if rendering user content in the webview, sanitize with `ammonia`.

```rust
// WRONG — shell injection via user_path
let output = std::process::Command::new("sh")
    .arg("-c")
    .arg(format!("ls -la {}", user_path))
    .output()?;

// RIGHT — each argument is a separate, uninterpreted value
let output = std::process::Command::new("ls")
    .arg("-la")
    .arg(&user_path)
    .output()?;
```

#### WebView Security

- **`dangerousRemoteDomainIpcAccess`**: never enable.
- **`withGlobalTauri`**: avoid in production. Use default module-based imports.
- **Navigation handler**: use `on_navigation` to block external URLs. Only allow `tauri://`, `http://localhost`, and `https://localhost`.

### Logging & Observability

- Use a Tauri logging plugin for structured logging. Configure per-module log levels. Log to both console and file.
- Integrate `sentry-tauri` for production crash reports and minidump collection. Install panic hooks to capture Rust panics before they terminate the process.
- Use the `tracing` crate with `tracing-subscriber` for structured, span-based observability.
- Add `#[tracing::instrument]` to all command handlers and service public methods.
- Generate a correlation UUID per frontend action and pass through IPC as a parameter. Log it as a span field for end-to-end tracing.
- Track command latency (P50/P95) and payload size in spans.
- Gate verbose spans behind `#[cfg(debug_assertions)]` or dynamic log level checks to prevent I/O bottlenecks in production.
- Profile before and after optimizations; avoid speculative tuning.

### Performance & Concurrency

**IPC throughput:**

- Prefer fewer, coarse-grained commands over chatty command chains. Batch related calls to reduce IPC overhead and frontend render churn.
- Return compact typed payloads; avoid oversized JSON responses.
- Paginate or chunk large lists and blobs. Keep payload schemas stable and explicit.

**Blocking work:**

- Move CPU-heavy or blocking work to `tokio::task::spawn_blocking`.
- Add timeouts for process and network calls.
- Avoid synchronous filesystem/process loops on async runtime threads.

**Shared state and locks:**

- Prefer immutable data flow and message-passing over shared mutable globals.
- Use `Arc<RwLock<T>>` only when read-heavy access justifies it.
- Keep lock scope minimal and never hold a lock across `.await`.
- Store dependencies in `AppState`; avoid hidden global state.

**Process management:**

- Reuse long-lived subprocesses where safe; avoid spawn-per-action designs.
- Persist and validate runtime process metadata (PID/port/path) on startup.
- Debounce file watcher and restart loops.

**Event emission:**

- Throttle or debounce high-frequency backend events.
- Coalesce bursts into deltas or periodic snapshots.
- Require listener cleanup in frontend hooks.
- Do not use events for one-off request/response operations.

**Startup, caching, and memory:**

- Cache expensive probes with explicit invalidation strategy.
- Defer non-critical scans until after first paint/ready state.
- Use bounded queues/channels to enforce backpressure.
- Avoid unnecessary `.clone()` of large data structures in hot paths.

## React Frontend

### Architecture & Directory Layout

Organize by **feature domain**, not by file type. Keep side-effects out of pure components.

```
src/
├── app/
│   ├── App.tsx
│   ├── routes.tsx
│   ├── providers.tsx        # QueryClient, Zustand, ErrorBoundary
│   └── error-boundary.tsx
├── features/
│   └── {name}/
│       ├── components/
│       ├── hooks/
│       ├── api/             # Typed invoke wrappers + Zod validation
│       ├── store.ts         # Zustand slice (if feature needs client state)
│       ├── schemas.ts       # Zod schemas for this feature
│       ├── types.ts
│       └── index.ts         # Public barrel — the ONLY import path
└── shared/
    ├── components/
    ├── hooks/
    ├── lib/
    │   ├── errors.ts        # AppError class + categorization
    │   └── tauri-query.ts   # TanStack Query helpers for Tauri
    ├── schemas/             # Shared Zod schemas
    └── types/
```

### Feature Boundary Rules

- **Dependency direction**: Features import from `shared/`, never from each other (components, hooks, utilities). If two features need to communicate, use shared state or Tauri events.
- **Type imports exception**: `import type` from another feature's barrel file is allowed for shared domain concepts (e.g., `import type { Workspace } from "@/features/workspaces"`).
- **Barrel file enforcement**: External code imports from `features/<name>` (the barrel `index.ts`), never from internal paths like `features/workspaces/hooks/useWorkspace`. Barrel files should be shallow (re-export only, no logic). Use `madge --circular` to detect circular imports.
- **Shared promotion**: When a type, hook, or component is used by more than one feature, promote it to `shared/` rather than creating cross-feature imports.
- Components should be presentational and receive data via props.
- Hooks handle data fetching, loading/error state, and orchestration.
- Components never call `invoke()` directly — they use feature API modules that wrap `invokeCommand`.

### State Management

Separate **server state** (data owned by the Rust backend) from **client state** (UI-only state). Do not mix them into a single store.

**Server State — TanStack Query:**

```tsx
export function useWorkspaces() {
  return useQuery({
    queryKey: ["workspaces"],
    queryFn: () => listWorkspaces(),
  });
}
```

- Use TanStack Query for all data from Tauri commands (handles caching, deduplication, background refetching, loading/error states).
- Invalidate queries via Tauri events: create a `useTauriEventInvalidation(event, queryKey)` hook that calls `listen()` in a `useEffect`, invalidates the query on event, and cleans up the listener on unmount.

**Client State — Zustand:**

```tsx
export const useWorkspaceUI = create<WorkspaceUIState>((set) => ({
  selectedId: null,
  select: (id) => set({ selectedId: id }),
  clear: () => set({ selectedId: null }),
}));
```

- Use Zustand for ephemeral UI state: sidebar open/closed, selected items, form drafts, theme preference.
- Never store server data in Zustand; use TanStack Query for that.
- Keep Zustand stores feature-scoped. Use separate stores per feature, not one global store.
- For state that must persist across app restarts, use a Tauri persistence plugin rather than Zustand persistence middleware.

### Error Handling

- Create a frontend `AppError` class in `shared/lib/errors.ts` that parses the backend's `{ kind, detail }` shape.
- Expose `isRetryable` (kind === "transient") and `isUserFixable` (kind === "validation") getters to drive UI behavior.
- Wrap the app root in an `ErrorBoundary` above `QueryClientProvider`.
- Place additional boundaries around each feature's top-level route component so one feature's failure doesn't crash the whole app.
- **Validation errors** → inline form feedback (field-level messages via Zod).
- **Not found** → empty state component with action (create, go back).
- **System errors** → toast notification with error detail.
- **Transient errors** → toast with retry button; TanStack Query's `retry` option for automatic retries.

### Performance & Rendering

- **React Compiler (default)**: React 19+ ships with the React Compiler. When enabled, skip manual memoization (`useCallback`, `useMemo`, `React.memo`) — the compiler handles reference stability automatically.
- **Without React Compiler**: stabilize references manually to prevent unnecessary re-renders from unstable function/object identity. Use `useCallback` for function props, or pass identifiers as props and let children create handlers.
- **Suspense waterfalls**: start independent async work in parallel (`Promise.all`). Avoid nested fetch initiation where children start only after parents resolve. Place Suspense boundaries so sibling areas resolve independently. In route loaders, fetch at the highest level that preserves ownership.
- **Deterministic rendering**: keep render functions pure (no side effects). Derive values from props/state; avoid mutable module-level UI state. Keep state local to the smallest useful subtree. Memoize only when profiling shows measurable benefit. Use React DevTools Profiler before and after optimizations.

## IPC & Type Sharing

### Type Strategy

| Type Category                                   | How Created                                 | Where Defined           | Zod Role                                             |
| ----------------------------------------------- | ------------------------------------------- | ----------------------- | ---------------------------------------------------- |
| IPC types (mirror Rust structs)                 | AI agents write manually                    | `features/*/types.ts`   | Runtime validation via `invokeCommand` catches drift |
| Frontend-only types (forms, URLs, localStorage) | Derive via `z.infer<typeof Schema>`         | `features/*/schemas.ts` | Source of truth for both type and validation         |
| State machine discriminated unions              | AI agents write manually (mirror Rust enum) | `features/*/types.ts`   | Optional runtime validation                          |

- Define each custom type once in a single canonical location. Never duplicate type definitions across files.
- Prefer extending or composing existing types (`Pick`, `Omit`, `Partial`, intersection) over creating near-identical copies.

### Typed Invoke Wrapper

All IPC calls go through a single `invokeCommand` utility:

```ts
import { invoke } from "@tauri-apps/api/core";
import { AppError } from "./errors";
import type { ZodSchema } from "zod";

export async function invokeCommand<T>(
  cmd: string,
  args: Record<string, unknown>,
  schema: ZodSchema<T>,
): Promise<T> {
  try {
    const data = await invoke<T>(cmd, args);
    return schema.parse(data);
  } catch (error) {
    throw AppError.fromBackend(cmd, error);
  }
}
```

For commands that return no data, use `z.void()`.

### Feature API Modules

Each feature wraps its IPC calls in an `api/` module:

```ts
import { invokeCommand } from "@/shared/lib/invoke";
import { z } from "zod";
import { WorkspaceSchema } from "../schemas";

export function listWorkspaces() {
  return invokeCommand("list_workspaces", {}, z.array(WorkspaceSchema));
}

export function createWorkspace(name: string, path: string) {
  return invokeCommand("create_workspace", { name, path }, WorkspaceSchema);
}

export function deleteWorkspace(id: string) {
  return invokeCommand("delete_workspace", { id }, z.void());
}
```

### Zod Validation

Define Zod schemas alongside feature types in `features/<name>/schemas.ts`. Shared schemas live in `shared/schemas/`. Validate all data crossing boundaries: IPC responses, form input, URL params, local storage reads.

```ts
export const WorkspaceSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  path: z.string(),
  createdAt: z.string().datetime(),
});
```

- Use Zod schemas with React Hook Form + `@hookform/resolvers/zod`. Define the schema, use `zodResolver(schema)` in `useForm`, and `formState.errors` provides field-level Zod messages automatically.
- Never use `as` type assertions to bypass validation — use `schema.parse()`.
- Never duplicate a Zod schema and a manual TypeScript type for the same data.
- Never validate inside render functions — validate in hooks, API modules, or event handlers. Exception: `schema.safeParse(value).success` for conditional rendering is acceptable.

## Conventions

### Naming

- All data validation functions must be named `validate...()` — in both Rust and TypeScript.
- Files containing JSX must use the `.tsx` extension. Files without JSX must use `.ts`.
- Never create `.js` or `.jsx` files. All frontend code must be TypeScript.

### Code Documentation

- Write comments for intent, constraints, invariants, or non-obvious tradeoffs — not narration.
- Remove or update comments whenever related code changes or is deleted.
- Refactor unclear code instead of adding explanatory comments.
- **Rust**: use `///` for all public items (include `# Errors`, `# Panics`, `# Safety` sections when applicable). Use `//!` at top of modules to explain why the module exists. Add `// SAFETY:` before every `unsafe` block. Use `//` for inline comments on non-obvious intent only.
- **TypeScript**: use `/** */` for all exported functions, components, hooks, types, and Zod schemas. Include `@param`, `@returns`, `@throws`, `@example` tags where they add clarity. Section comments are for visual grouping only — JSDoc takes priority for API documentation.
- **Section comments** (same format in both Rust and TypeScript):

```
// ----
//
// Section Header (h1)
//
// ----

//
// Subsection (h2)
// ----

//
// Minor heading (h3)

// Regular comment (p)
```

## Testing & Tooling

### Testing Policy

Use risk-based testing. Optimize for defect detection, not test count. Test behavior that can break users, data integrity, security, or release safety. Do not test framework internals, trivial pass-through wrappers, or static markup with no logic. Every test should map to a named risk.

**Required test mix:**

- Rust unit tests: domain logic, invariants, parsers, error mapping.
- Rust command integration tests: critical commands with happy + failure path.
- IPC contract tests: payload shape/version compatibility across Rust and TypeScript.
- Frontend tests: hooks/state transitions (`idle/loading/success/error`) and critical UI behavior.
- Zod schema tests: validate schemas match actual backend response shapes (catches drift).
- E2E tests: only top user journeys and one failure-recovery journey per critical feature.
- Cross-platform smoke: app starts and primary flow works on macOS/Linux/Windows.

**Anti-theater guardrails:**

- Coverage is a secondary metric; do not chase arbitrary percentages.
- Prefer focused assertions over snapshots. Delete tests that no longer protect meaningful risk.
- Fix or remove flaky tests immediately; do not normalize retries as success.
- Required coverage: critical paths (authentication, data persistence, state machine transitions, error mapping). Optional: UI rendering, config loading, simple CRUD pass-throughs. Use coverage reports to find untested critical paths, not to chase percentages.

**Per-feature test protocol:**

- One short risk list (`data-loss`, `security`, `workflow-break`, `perf-regression`).
- Minimum tests that prove those risks are controlled.
- One explicit statement of what is intentionally not tested and why.
- Stop when highest risks are covered with deterministic, maintainable tests.

### Dev Commands

| Command                        | Purpose                                             |
| ------------------------------ | --------------------------------------------------- |
| `bun install`                  | Install dependencies                                |
| `bun tauri dev`                | Dev mode (hot-reload frontend + Rust recompilation) |
| `bun run lint`                 | Lint frontend                                       |
| `bun test`                     | Run frontend tests                                  |
| `cd src-tauri && cargo test`   | Run Rust tests                                      |
| `cd src-tauri && cargo clippy` | Lint Rust                                           |

- **Build order**: `bun install` → `bun run build` → `bun tauri build`.

### Test Tooling

| Layer                 | Tools                                       | Purpose                                                      |
| --------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| Rust unit/integration | `rstest` + `mockall`                        | Parameterized tests, trait mocking for repositories/services |
| Frontend unit         | `vitest` + `@testing-library/react` + `msw` | Component/hook tests with mocked IPC via MSW                 |
| E2E                   | `tauri-driver`                              | WebDriver-based end-to-end tests against a built app         |
| IPC contract          | Zod schema contract tests                   | Detect drift between Rust types and frontend schemas         |

- IPC contract testing: CI starts the backend, invokes each command with fixture data, and validates responses against the Zod schemas. Fail the build on shape mismatches.
- CI must run tests on macOS, Linux, and Windows runners.
- Use `tauri-driver` for headless smoke tests: app launches, primary navigation works, no crash on startup.
- Minimum cross-platform bar: app opens, displays main view, and responds to one navigation action without error.

## Standards

- Read 2+ existing files of the same type before creating new ones — match structure exactly
- Commands are thin IPC wrappers with no business logic; services hold domain logic
- All IPC responses validated by Zod schemas via `invokeCommand`
- State machine transitions validated in services, not command handlers
- Error variants carry enough context for frontend display (`{ kind, detail }`)
- Frontend organized by feature domain; features never import from each other (except `import type`)
- Server state in TanStack Query, client state in Zustand — never mix
- Prefer official Tauri plugins before custom Rust commands for common desktop tasks (settings persistence, window state, native dialogs, logging, secret storage). Use custom implementation only when no plugin covers the requirement, the plugin cannot meet a hard security/performance requirement, or a custom flow is required and the reason is documented
- Risk-based testing: every test maps to a named risk (data-loss, security, workflow-break, perf-regression)
- Document state machine transition tables in service module rustdoc
- Every public service method documents its `# Errors` section in rustdoc
- IPC contracts defined by the feature's `api/index.ts` — its signatures and return types are the handoff point
- Document non-obvious service dependencies in the module's `//!` doc comment
- Use `///` for all public Rust items; `/** */` for all exported TS functions/types
- Add `#[tracing::instrument]` to all command handlers and public service methods

## Constraints

- Never use `unwrap()` or `expect()` in `commands/` or `services/` — use `?` or explicit handling
- Never use `unsafe-eval` or `unsafe-inline` in CSP
- Never enable `dangerousRemoteDomainIpcAccess` or `withGlobalTauri` in production
- Never interpolate user input into shell commands — use `Command::new().arg()` per argument
- Never store secrets in config files, localStorage, or serialized state — use OS keychain
- Never hold a lock across `.await` boundaries
- Never create `.js`/`.jsx` files — all frontend code is TypeScript
- Never store server data in Zustand — use TanStack Query
- Never use `as` type assertions to bypass Zod validation — use `schema.parse()`
- Never duplicate type definitions — define once in canonical location
- Components never call `invoke()` directly — use feature API modules
- Never over-abstract: three similar lines are better than a premature abstraction
- Never create separate files for single-use types or tiny helpers
- Never mix naming conventions across files (e.g., `getWorkspaces` in one feature, `fetchProjects` in another)
- Never add caching, memoization, or batching before measuring a performance problem
- Never swallow errors silently — every caught error is re-thrown, returned as typed error, or logged with context
- Use `HashRouter`, not `BrowserRouter`
- Use `tauri::async_runtime::spawn()`, not `tokio::spawn()`
- No excessive `.clone()` instead of borrowing in hot paths
- No giant `match` blocks where traits should model polymorphism
- No stringly-typed domain values where enums fit better
- No monolithic `main.rs` with business logic
- No overusing `Arc<Mutex<T>>` for shared state
- No deferring platform-specific abstractions until late
- No prop drilling through multiple component levels
- No cascading `useEffect` chains
- No mixing data fetching with presentational rendering
- No ignoring loading/error states for async operations
- No inconsistent error shapes across backend and frontend
- No testing only the happy path — model failure modes explicitly
- No hardcoding paths — use platform-appropriate path APIs
