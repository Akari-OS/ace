# Changelog

All notable changes to the ACE (Agent Context Engineering Framework) specification are
documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and ACE follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html) at the spec level.

## [v0.1.0-draft] — 2026-04-14

Initial public draft of the ACE Framework. See [`spec/v0.1/protocol.md`](spec/v0.1/protocol.md).

### Added

- **7-layer reference model.** Positions ACE across the agent context stack, pointing to
  MCP (Layer 3), AMP (Layer 4), and M2C (Layer 1) for adjacent concerns. Layers 2 (Context
  Files) and 5 (Access Control) are normatively defined by ACE.
- **`.acp-ignore` (Layer 5).** File-format spec, pattern syntax, default-deny secret list,
  `allow:` override, `bypass:` with human approver + expiry, audit requirements.
- **Context file frontmatter (Layer 2).** Minimum required fields (`spec-version`,
  `applies-to`, `activation`), recommended fields, vendor-extension namespace, and the
  four-mode activation model (`always` / `auto-attached` / `agent-requested` / `manual`).
- **Lint rules.** Two-tier model: structural lint (regex/AST, <100 ms/file) and optional
  semantic lint (LLM-powered, PR-time only). Severity model + SARIF 2.1.0 output.
- **Adapter layer.** Read existing `AGENTS.md` / `CLAUDE.md` / `.cursor/rules` /
  `copilot-instructions.md` / Windsurf rules without requiring rewrite.
- **Security considerations.** Rules File Backdoor mitigation, Unicode Tag / ZWSP / BiDi
  detection, supply-chain hardening via Sigstore signing.
- **Audit & provenance.** SARIF 2.1.0 reports, Sigstore signing, optional SBOM
  (SPDX / CycloneDX).

### Governance

- Initial BDFL under Akari-OS. Transition to a neutral foundation (LF sub-foundation
  candidate) planned by v1.0.
- DCO adopted for contributions (no CLA).
- Spec licensed under CC-BY 4.0; reference implementation under Apache 2.0.

### Unreleased / Planned

- `v0.2` — Semantic lint ruleset stabilization; multi-language (ja/en) frontmatter.
- `v0.3` — Enterprise extensions (SBOM / SLSA Level 3 / 90-day patch SLA).
- `v1.0` — Two-independent-implementation interop demonstrated; neutral foundation
  transfer initiated.
