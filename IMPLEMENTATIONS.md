# ACE — Known Implementations

This document lists known ACE implementations, adapters, and integrations.

## Reference Implementations

Currently none. ACE v0.1 is in draft.

The first reference implementation (`ace` — Rust CLI + LSP, under Apache 2.0) is planned
under the Akari-OS organization. It will ship alongside v0.1 spec stabilization and
target:

- `ace lint` — structural + semantic context-file lint
- `ace check` — `.acp-ignore` respect check for a given command
- `ace serve` — LSP mode for editor integration
- `ace audit` — SARIF 2.1.0 report generation
- GitHub Action wrapper

## Community Providers

None yet. Contributions welcome.

## Adapters (Planned)

Adapters read existing context files and expose them through the ACE internal
representation, enabling lint & audit without requiring rewrite.

| Source format | Status |
|---|---|
| `AGENTS.md` (AAIF) | Planned v0.1 |
| `CLAUDE.md` (Anthropic Claude Code) | Planned v0.1 |
| `.cursor/rules/*.mdc` | Planned v0.1 |
| `.github/copilot-instructions.md` | Planned v0.2 |
| Windsurf rules (`.windsurfrules`) | Planned v0.2 |
| Cline rules + Memory Bank | Planned v0.2 |
| Claude Code `SKILL.md` | Planned v0.2 |

## How to Add Your Implementation

Submit a PR adding your implementation to the table below. Include:

- Name / Link
- ACE version supported (v0.1 / ...)
- Language / Runtime
- Role (linter / adapter / integration / host tool)
- License
- Status (alpha / beta / production)

## Planned Implementations

| Name | Language | Planned ACE Version | Role | Notes |
|---|---|---|---|---|
| `ace` (reference CLI + LSP) | Rust | v0.1 | linter + adapter | Apache 2.0, ships under Akari-OS |
| AkariPool `.acp-ignore` guard | Rust | v0.1 | host tool | Blocks analyzer access to deny-listed paths |
| AKARI Video context view | TypeScript | v0.1 | integration | Reads Pool items through ACE adapter |

## Inspiration

- [AGENTS.md / AAIF](https://agents.md) — cross-vendor root file standard
- [Cursor `.cursor/rules`](https://docs.cursor.com/context/rules-for-ai) — 4-mode activation model
- [Claude Code Skills](https://github.com/anthropics/claude-skills) — progressive disclosure
- [Vale](https://vale.sh/) — engine-less linter architecture
- [Ruff](https://docs.astral.sh/ruff/) — Rust linter performance ceiling
