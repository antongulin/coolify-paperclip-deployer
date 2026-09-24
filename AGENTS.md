# Repository instructions

`AGENTS.md` is the only canonical repository instruction source. Do not add client-specific
instruction copies (`CLAUDE.md`, `GEMINI.md`, `.cursorrules`, or similar); client adapters must
point at this file, not fork it. This file follows the [DOX](https://github.com/agent0ai/dox)
AGENTS.md framework (Markdown guidance only; no runtime), integrated from upstream revision
`765ae4ac02cc884eefcd41a3d0f71941721adb89` (MIT).

The closest `AGENTS.md` is the binding work contract for its subtree. Before editing, read this
root file and every child `AGENTS.md` along the target path. After a meaningful change, update the
nearest owning instructions and affected parent index; remove stale or contradictory guidance.

## Session Git rules

Start every session by inspecting Git status and worktrees, fetching origin with pruning, safely fast-forwarding local main, and verifying main matches origin/main. Only then create a task branch from synchronized main if needed. Preserve existing task branches and unfinished work. Never reset, discard changes, auto-stash, or force-push merely to synchronize. If safe synchronization is blocked, resolve the blocker before editing or branching.

## Repository layout

```
paperclip-coolify-deployer/SKILL.md     -> the OpenCode-compatible deployer skill (agent instructions)
paperclip-coolify-deployer/benchmark.json, evals/evals.json -> skill benchmark + eval cases (data)
GUIDE.md                                -> human step-by-step Paperclip-on-Coolify install guide
README.md, reddit-post-rselfhosted.md, LICENSE -> overview, launch post, MIT license
.github/workflows/code-review.yml      -> reusable Robin PR-review workflow (PR/issue_comment only)
.mcp.json, opencode.jsonc, .codex/, .cursor/, .vscode/ -> committed CodeGraph MCP wiring per client
.codegraph/                            -> machine-local CodeGraph index (untracked, gitignored)
```

## Code intelligence: CodeGraph

- **CodeGraph (`codegraph`) is the approved code-intelligence index.** Project-local MCP configs are committed for Claude-compatible [`.mcp.json`](.mcp.json), Codex [`.codex/config.toml`](.codex/config.toml), OpenCode [`opencode.jsonc`](opencode.jsonc), Cursor [`.cursor/mcp.json`](.cursor/mcp.json), and VS Code [`.vscode/mcp.json`](.vscode/mcp.json). Each launches `codegraph serve --mcp`, sets `CODEGRAPH_TELEMETRY=0`, and passes `--path ${workspaceFolder}` where the client supports a workspace placeholder. Clients without a workspace placeholder (`.mcp.json`, `.codex/config.toml`, `opencode.jsonc`) rely on the MCP client's project root/`rootUri` instead, so start those clients from this checkout and confirm `codegraph status` reports this project — a CLI run from a parent directory targets that parent. Do not let an installer create a competing instruction file, and do not add or edit home-directory configs from this repo.
- **Telemetry is off** via the committed `CODEGRAPH_TELEMETRY=0`. Keep it off.
- **The index stays untracked.** `.codegraph/` is gitignored and must never be committed. Build it once per checkout, then check it:

  ```bash
  CODEGRAPH_TELEMETRY=0 codegraph init .
  CODEGRAPH_TELEMETRY=0 codegraph sync .   # after pulls or merges that add files
  CODEGRAPH_TELEMETRY=0 codegraph status   # confirm "Index is up to date"
  ```

- **Honest coverage: this repository has no executable source.** Its content is Markdown (`paperclip-coolify-deployer/SKILL.md`, `GUIDE.md`, `README.md`, `reddit-post-rselfhosted.md`), JSON data (`benchmark.json`, `evals/evals.json`), and one CI workflow YAML. There is no `.py`, `.js`, `.ts`, or other symbol-bearing code, so CodeGraph indexes **zero symbols** here — expect an empty `query` result. Treat CodeGraph purely as a file lister for this repo and use ordinary search and read for all navigating and editing; verify every finding against the actual file.
- **Canonical source:** `https://github.com/colbymchenry/codegraph`. The global `codegraph` binary is a user-managed, pre-existing install (ask-first to add or update); per-repo setup here is only the wiring plus the local index. Confirm `command -v codegraph` and `codegraph version` before relying on it.
- **Commands:** `codegraph status`, `codegraph query "<symbol>"`, `codegraph node <file-or-symbol>`, `codegraph explore "<area>"`, `codegraph files`.

## Boundaries

### Always

- Keep `AGENTS.md` the only canonical repository instruction file.
- Follow the shared workflow's change classification and Git route; use atomic conventional commits.

### Ask first

- Add dependencies, change CI/release/visibility, or install a skill.
- Add or update the global `codegraph` binary.

### Never

- Commit `.codegraph/` (local, gitignored index state) or credentials/secrets.
- Add a competing root instruction file or edit home-directory configs.
- Merge with unresolved review threads. Follow the shared workflow and explicit user authorization
  for the publication route, including any direct-main exception.

## Child DOX Index

No child `AGENTS.md` files exist. Add and index one when a folder becomes a durable ownership
boundary with distinct instructions; keep this root contract current.
