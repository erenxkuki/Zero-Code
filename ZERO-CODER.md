# Zero Coder — fork deltas on top of OpenCode

Zero Coder keeps the OpenCode engine (session runner, providers, tools, server)
and layers product branding plus three features on top. This file lists every
intentional delta so upstream merges stay reviewable.

Layout: engine packages live in `packages/` at the repo root; the Tauri 2
desktop shell (Rust backend + React frontend) lives in `shell/` with its own
`package.json` and `README.md`.

## Branding (display surface only)

| Area | Change |
|---|---|
| Desktop installer | `productName` → `Zero Coder [Dev/Beta]`, `artifactName` → `zero-coder-desktop-*`, app id → `ai.zerocoder.desktop[.dev/.beta]`, URL scheme → `zero-coder://`, rpm names → `zero-coder*` (`packages/desktop/electron-builder.config.ts`) |
| Web/desktop titles | `<title>Zero Coder</title>` (`packages/desktop/src/renderer/index.html`, `packages/app/index.html`) |
| CLI | `zero-coder` bin alias ships the same launcher (`packages/opencode/package.json`); help/`scriptName` follows the invoked binary name (`src/index.ts`); user-facing copy says Zero Coder / `zero-coder …` (`src/cli/error.ts`, `src/cli/cmd/attach.ts`) |

Deliberately **not** renamed (functional compat): npm package names
(`@opencode-ai/*`), `OPENCODE_*` env vars, config/data dirs, `opencode.json`
filename, `opencode.local` mDNS default, updater publish targets, models.dev
`USER_AGENT`, TUI ASCII logo art, `opencode/*` provider ids.

## Feature 1 — Universal Provider Gateway

Custom endpoints already work via `options.baseURL` + `models` in
`opencode.json` (generic `openai-compatible` adapter). Zero Coder adds dynamic
discovery so the model list is never hand-maintained:

```sh
zero-coder discover http://localhost:11434/v1 --provider-id ollama-local
# prints `provider/<model>` lines + a ready-to-paste opencode.json block
```

(`packages/opencode/src/cli/cmd/discover.ts`; `GET <base>/models`, 15s timeout,
optional `--api-key`, `--adapter` defaults to `@ai-sdk/openai-compatible`.)

## Feature 2 — Auto-Anchor Context Engine (85%)

`compactIfNeeded` now compacts when estimated usage passes
`min(context − max(output, buffer), context × anchorThreshold)` with
`anchorThreshold` defaulting to **0.85** (configurable via
`compaction.anchorThreshold`, snapshot dir via `compaction.anchorDir`,
default `.zero`).

On every successful compaction the runner writes a durable snapshot with the
compaction summary + retained recent context:

`.zero/anchor.json` → `{ version, sessionID, createdAt, summary, recent }`

- Writer: `SessionAnchor` (`packages/core/src/session/anchor.ts`), pure
  `build` + `Effect` `write`/`read`, `writeBestEffort` (self-provided
  `NodeFileSystem`, errors swallowed — an anchor can never fail a compaction).
- Hook: `onSummary` callback threaded through `SessionCompaction` input from
  the runner (`session/runner/llm.ts` passes `session.location.directory`).
- Inspect: `zero-coder anchor` prints the snapshot summary for the cwd,
  `zero-coder anchor --json` prints raw JSON, `zero-coder anchor --resume`
  prints a ready-to-paste resumption prompt for a new session.

## Feel (why it beats stock opencode)

- Shell chat, provider settings, and token gauge persist across restarts
  (`zustand/persist`, last 100 messages, ephemeral streaming state excluded).
- Agent patch blocks go to an **Apply / Dismiss** review card with a unified
  diff preview — never silently auto-applied.
- Shell slash commands: `/fix` (repair from errors), `/test` (suite to green),
  `/anchor` (recap session), `/clear` (reset chat).
- Chat auto-scrolls; the token gauge follows the selected model's context
  window when the endpoint reports one.
- Shell preview has one-click **Send errors to agent**, drafting flagged
  stderr lines into the composer for the fix loop.

## Feature 3 — Live Preview + self-healing hook

```sh
zero-coder preview "npm run dev" --port 5173 [--cwd <dir>]
```

Spawns the dev server, prefixes streamed lines (`[preview:out|err]`),
flags error lines (`error|failed|cannot|exception|EADDRINUSE|ERR_*`),
forwards `SIGINT`, propagates the exit code, and tells the user to paste
flagged lines back into the session so the agent writes a fix.
(`packages/opencode/src/cli/cmd/preview.ts`.)
The Tauri shell closes the loop in-app: `shell/src/components/LivePreview.tsx`
collects flagged error lines and drafts them into the chat composer via
`draftPrompt` (`useAgentStore`) with one click — no copy-paste.

## Feature 4 — Fuzzy edit fallback (engine)

The core `edit` tool was exact-match-only with an explicit TODO to port fuzzy
strategies. Zero Coder implements the fallback (`packages/core/src/tool/edit.ts`):
when `oldString` has zero exact matches, an indentation-insensitive sliding-window
matcher (trimmed-line equality over non-blank lines, ≥80% agreement) applies the
replacement and marks output `fuzzy: true` (additive optional field, surfaced as
`(fuzzy line match)` in model output). Zero matches still fail; multiple
approximate spans fail asking for more context. Pure matcher covered by
`packages/core/test/tool-edit-fuzzy.test.ts`.

## Verify (needs bun toolchain — not available on this machine)

```sh
bun install
bun typecheck            # per-package: packages/opencode, packages/core, …
bun test --timeout 30000 # from package dirs, never repo root
bun run ./packages/opencode/src/index.ts discover http://localhost:11434/v1
```
