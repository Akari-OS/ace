# ACE — Agent Context Engineering Framework

> Standardize how AI agents read, respect, and reason about your code.
>
> 📜 **Type: Specification** — This repo holds the protocol spec only. A reference implementation is planned for Phase C. See [Known Implementations](#known-implementations).

## What is ACE?

ACE is an open protocol framework that standardizes **what AI agents can read, how context files are structured, and how they're validated**. Like MCP standardized tool calling and AMP standardized memory, ACE standardizes the context layer.

It defines:

1. **Access Control** — `.acp-ignore` as the AI-era equivalent of `robots.txt` + `.gitignore`
2. **Context File Spec** — Frontmatter conventions unifying `AGENTS.md` / `CLAUDE.md` / `.cursor/rules`
3. **Lint Rules** — Static + LLM-powered validation for context quality
4. **Adapter Layer** — Read existing `CLAUDE.md` / `AGENTS.md` / Cursor rules without rewrite
5. **Audit / Provenance** — SARIF-compatible reports + Sigstore signatures

```
Agent → [ACE Linter / Adapter] → Your repo
              ↕
        .acp-ignore (access contract)
              ↕
        Context files (frontmatter + lint)
```

## Why?

AI agents today...

- **read your `.env` files** unless you block them — and `.gitignore` does not block AI
- **see contradictory rules** across `CLAUDE.md`, `AGENTS.md`, `.cursor/rules`, Copilot instructions
- **are exploitable** via invisible-Unicode prompt injection in context files (Rules File Backdoor, success rate 85%+)
- **each vendor reinvents** the same glob / activation / frontmatter concepts, incompatibly

**Before ACE**: `.env` quietly read. `CLAUDE.md` and `AGENTS.md` drift apart. Rules File Backdoor trivially exploitable. Every vendor ships its own bespoke format.

**After ACE**: Typed, linted, vendor-neutral context. A single access contract (`.acp-ignore`) every compliant tool must respect. Existing files adopted by adapters — no rewrite required.

### Where ACE fits

```
MCP  → Standardizes tool calling  (what agents CAN DO)
M2C  → Standardizes media context (what agents UNDERSTAND)
AMP  → Standardizes memory        (what agents REMEMBER)
ACE  → Standardizes context       (what agents READ)     ← NEW
A2A  → Standardizes communication (how agents TALK)
```

No existing standard covers the **access + validation** layer for AI-consumable context. ACE fills this gap.

## The killer feature: `.acp-ignore`

`.acp-ignore` is the AI-era equivalent of `robots.txt` + `.gitignore`, fused. A single file at repo root declaring **what AI agents may and may not read** — with a default-deny list for secrets and a signed audit trail for violations.

```gitignore
# .acp-ignore — ACE v0.1

# === Default deny (shipped with ACE) ===
.env
.env.*
*.key
*.pem
id_rsa*
.aws/credentials
.ssh/**
secrets/**

# === Repo-specific deny ===
deny: internal/legal/**
deny: **/*.customer-data.json

# === Explicit allow (overrides default deny) ===
allow: .env.example
allow: docs/public-keys/*.pem

# === Bypass (requires human approval + expiry) ===
bypass: debug/live-session.log
  approver: Akari-OS maintainers
  expires: 2026-05-01
  reason: live incident triage
```

Compliant tools MUST:

- Refuse to read any path matching `deny`
- Emit a `violation` event to `audit.log` when a read attempt is blocked
- Require an explicit `bypass:` entry (with approver + expiry) before granting temporary access

## Architecture — 7-layer Reference Model

ACE positions itself as a **reference model** across the agent context stack, defining Layer 2 and Layer 5 normatively and pointing to sibling protocols elsewhere.

```
┌──────────────────────────────────────────────────────┐
│ Layer 6: Orchestration                               │
│   Spec Kit / Harmony / Autopilot frameworks          │
├──────────────────────────────────────────────────────┤
│ Layer 5: Access Control        ⭐ ACE core            │
│   .acp-ignore / Capability decl / Audit provenance   │
├──────────────────────────────────────────────────────┤
│ Layer 4: Memory                                      │
│   → AMP (Agent Memory Protocol)                      │
├──────────────────────────────────────────────────────┤
│ Layer 3: Tools                                       │
│   → MCP (Model Context Protocol)                     │
├──────────────────────────────────────────────────────┤
│ Layer 2: Context Files         ⭐ ACE core            │
│   Frontmatter spec / Lint CI / Adapter layer         │
├──────────────────────────────────────────────────────┤
│ Layer 1: Media Semantics                             │
│   → M2C (Media-to-Context)                           │
├──────────────────────────────────────────────────────┤
│ Layer 0: Raw Data                                    │
│   Pool / File system / DB                            │
└──────────────────────────────────────────────────────┘
```

- **ACE defines** Layers 2 and 5.
- **ACE points to** existing protocols (MCP / AMP / M2C) for Layers 1, 3, 4.
- **ACE does not compete** with Spec Kit (Layer 6) or MCP (Layer 3) — it complements them.

## Specification

**Latest:** v0.1.1

```
spec/
└── v0.1/
    └── protocol.md      ← Core spec (draft)
```

The v0.1 spec is intentionally narrow: `.acp-ignore` + frontmatter + minimal lint + adapter hints. Semantic (LLM-powered) lint and enterprise features (SBOM, SLSA) land in later minor versions.

## Design Principles

| Principle | Description |
|---|---|
| **Vendor-neutral** | No lock-in to Cursor / Claude / Copilot / Windsurf. Adapters bridge each. |
| **Security-first** | Default-deny secret list shipped in v0.1. Rules File Backdoor mitigations are normative, not optional. |
| **Tailwind pattern** | Light entry (`.acp-ignore` alone in 5 minutes) → deep adoption (full lint CI + provenance) over time. |
| **Adapter-based** | Don't replace existing files. Read `AGENTS.md` / `CLAUDE.md` / Cursor rules where they live. |
| **Spec vs code split** | Prose under CC-BY 4.0, reference implementation under Apache 2.0 (OpenAPI / Schema.org pattern). |

## Known Implementations

See [IMPLEMENTATIONS.md](./IMPLEMENTATIONS.md). Reference implementation (Rust, CLI + LSP) is planned alongside v0.1 spec stabilization.

## Related

- [AMP — Agent Memory Protocol](https://github.com/Akari-OS/amp)
- [M2C — Media to Context](https://github.com/Akari-OS/m2c)
- [AKARI-OS](https://github.com/Akari-OS) — umbrella organization

## Governance

Currently **BDFL under Akari-OS**. Explicit plan to transition to a neutral foundation (LF sub-foundation, sibling to MCP / A2A / x402) by v1.0 — contingent on two independent implementations, a third-party security audit, and 2–3 production adopters outside AKARI.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the RFC / PR process.

## License

- **Specification (prose)**: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- **Reference implementation (code)**: [Apache 2.0](./LICENSE)
- **Examples**: CC0 / MIT (per-file)
- **Contributions**: [DCO](https://developercertificate.org/) (`Signed-off-by` on every commit)

See [TRADEMARKS.md](./TRADEMARKS.md) for the "ACE" / "Agent Context Engineering" name policy.

## Inspiration

- [MCP (Model Context Protocol)](https://modelcontextprotocol.io) — tool calling standard
- [AGENTS.md (AAIF)](https://agents.md) — cross-vendor context file spec
- [robots.txt](https://www.robotstxt.org/) — web crawler access convention
- [SARIF 2.1.0](https://docs.oasis-open.org/sarif/sarif/v2.1.0/) — static analysis result format
- [Sigstore](https://www.sigstore.dev/) — supply-chain signing
