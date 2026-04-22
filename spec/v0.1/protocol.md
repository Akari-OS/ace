---
spec-id: ACE-001
version: 0.1.1
status: draft
created: 2026-04-14
updated: 2026-04-22
ai-context: claude-code
related-specs:
  - AMP v0.1 (github.com/Akari-OS/amp)
  - M2C v0.2 (github.com/Akari-OS/m2c)
---

# ACE Framework Specification — v0.1 (draft)

> **ACE** — Agent Context Engineering Framework.
> Standardizes how AI agents read, respect, and reason about a repository's context
> files. This document is the normative specification for ACE v0.1 (draft).

---

## 0. Abstract

AI coding agents consume a repository's files, configuration, and natural-language
instructions to produce code and advice. Today, every agent framework (Claude Code,
Cursor, Copilot, Windsurf, Cline, Aider, …) reinvents the same three concerns with
incompatible surfaces:

1. **Access control** — which files the agent may read
2. **Context file format** — how instructions are encoded (frontmatter, activation,
   scoping)
3. **Validation** — how stale, contradictory, or malicious context is detected

ACE defines a minimal, vendor-neutral contract for all three, plus an **adapter layer**
that lets existing files (`AGENTS.md`, `CLAUDE.md`, `.cursor/rules/*.mdc`,
`copilot-instructions.md`, Windsurf rules) participate without rewrite. ACE does not
replace existing root-file conventions; it wraps them.

v0.1 is intentionally narrow: `.acp-ignore` + a frontmatter schema + a structural lint
surface + adapter hints. Semantic (LLM-powered) lint, enterprise provenance features
(SBOM / SLSA Level 3 / 90-day patch SLA), and signed certified-conformance testing are
deferred to later versions.

---

## 1. Conformance

### 1.1 Key words

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document
are to be interpreted as described in [RFC 2119] and [RFC 8174], and only when appearing
in ALL CAPITALS.

[RFC 2119]: https://www.rfc-editor.org/info/rfc2119
[RFC 8174]: https://www.rfc-editor.org/info/rfc8174

### 1.2 Actors

- **Host tool** — the agent framework or IDE extension reading files on behalf of an AI
  model (e.g., Claude Code, Cursor, a custom CLI agent).
- **Linter** — an implementation that validates a repository's context files against
  ACE rules and emits findings.
- **Adapter** — a component that reads a non-ACE-native file format (e.g., `AGENTS.md`)
  and exposes it through ACE's internal representation.
- **Author** — the human (or AI agent authoring PRs) who writes and maintains context
  files.

### 1.3 Conformance levels

An implementation MAY claim ACE conformance at one of the following levels. All claims
MUST specify the level and the ACE spec version (e.g., "ACE v0.1 L1").

- **L1 — Access Control**: Honors `.acp-ignore` (§3). Sufficient for most host tools.
- **L2 — Lint**: In addition, implements the structural lint surface (§5) and emits
  SARIF 2.1.0 output (§5.4).
- **L3 — Full**: In addition, implements at least one adapter (§6) and emits provenance
  (§7).

### 1.4 Out of scope (v0.1)

- Prescribing which LLM performs semantic lint
- Prescribing an enterprise patch SLA
- Certified-conformance testing suites
- Localization of lint messages beyond English + Japanese
- Runtime agent-to-agent negotiation of access control (see Layer 6 — orchestration)

---

## 2. Architecture — 7-Layer Reference Model

ACE is best understood as a layered reference model spanning the AI agent context
stack. ACE normatively defines Layer 2 and Layer 5; other layers are addressed by
sibling protocols and are listed here for orientation only.

```
┌──────────────────────────────────────────────────────┐
│ Layer 6: Orchestration                               │
│   Spec Kit / Harmony / Autopilot frameworks          │
├──────────────────────────────────────────────────────┤
│ Layer 5: Access Control        ⭐ defined in §3       │
│   .acp-ignore / capability decl / audit provenance   │
├──────────────────────────────────────────────────────┤
│ Layer 4: Memory                                      │
│   → AMP (Agent Memory Protocol)                      │
├──────────────────────────────────────────────────────┤
│ Layer 3: Tools                                       │
│   → MCP (Model Context Protocol)                     │
├──────────────────────────────────────────────────────┤
│ Layer 2: Context Files         ⭐ defined in §4 / §5  │
│   frontmatter / lint / adapter                       │
├──────────────────────────────────────────────────────┤
│ Layer 1: Media Semantics                             │
│   → M2C (Media-to-Context)                           │
├──────────────────────────────────────────────────────┤
│ Layer 0: Raw Data                                    │
│   Pool / file system / DB                            │
└──────────────────────────────────────────────────────┘
```

- **Layer 2 (Context Files)** defines the frontmatter schema, activation semantics, and
  status lifecycle common to all ACE context files.
- **Layer 5 (Access Control)** defines `.acp-ignore`, the audit contract, and bypass
  discipline.
- Layers 1, 3, 4, 6 are referenced but not defined by this document.

---

## 3. Layer 5 — Access Control (`.acp-ignore`)

### 3.1 File format

- A repository MAY contain exactly one `.acp-ignore` file at the repository root.
- The file MUST be UTF-8 encoded.
- Line endings MAY be LF or CRLF; implementations MUST accept both.
- Maximum file size: 128 KiB. Implementations MUST refuse files larger than this and
  emit a `violation.acp-ignore-too-large` finding.

### 3.2 Pattern syntax

Patterns follow `.gitignore` glob semantics with the following clarifications:

- `*` matches any sequence of non-`/` characters
- `**` matches any sequence including `/`
- A leading `/` anchors to the repo root
- A trailing `/` restricts the match to directories
- `#` begins a line comment; inline comments are NOT supported
- Whitespace-only lines are ignored

The following directive prefixes are recognized (one per line, outside comments):

| Prefix | Meaning |
|---|---|
| (no prefix) | Implicit `deny:` (legacy `.gitignore` compatibility) |
| `deny: <pattern>` | Deny read access to paths matching the pattern |
| `allow: <pattern>` | Override prior deny (including default-deny) for the pattern |
| `bypass: <pattern>` | Time-limited deny override; MUST include `approver` + `expires` |

A `bypass:` block begins on the line containing `bypass: <pattern>` and ends at the
first line that is either (a) empty (zero characters after trimming whitespace) or
(b) a new top-level directive (`deny:`, `allow:`, `bypass:`, or a bare pattern with no
leading whitespace). Continuation lines MUST be indented by at least one space or tab.
Implementations MUST reject a `bypass:` block that lacks both `approver:` and `expires:`
before the block ends.

Example of a well-formed `bypass:` block:

```
bypass: debug/trace.log
  approver: @Akari-OS/maintainers
  expires: 2026-10-01
  reason: Temporary trace access for incident #42

deny: internal/**
```

Within a `bypass:` block, the following sub-fields are recognized:

- `approver: <identity>` — REQUIRED. An identity handle (`@github-user`, email, or
  opaque string). Implementations MUST record this in the audit log.
- `expires: <YYYY-MM-DD>` — REQUIRED. ISO 8601 date. After this date, the bypass MUST
  NOT be honored and implementations MUST emit a `violation.acp-ignore-expired-bypass`
  finding.
- `reason: <free text>` — OPTIONAL but RECOMMENDED. One line.

### 3.3 Default deny list

Compliant host tools MUST treat the following patterns as implicitly denied even when
`.acp-ignore` is absent:

```
.env
.env.*
*.key
*.pem
*.p12
*.pfx
id_rsa*
id_ed25519*
*.gpg
.aws/credentials
.aws/config
.ssh/**
.netrc
.npmrc
.pypirc
secrets/**
credentials/**
private/**
```

This list is frozen for v0.1 and MAY only be extended (not reduced) in subsequent
minor versions.

### 3.4 Allow override

An explicit `allow: <pattern>` MUST override both the default-deny list and any
repo-specific `deny:` for matching paths. `allow:` is evaluated after all `deny:`
directives regardless of file order.

When multiple `allow:` entries match the same path, the **most-specific pattern wins**,
where specificity is defined as the number of non-wildcard path segments. If two
patterns have equal specificity, the **last** matching `allow:` in file order wins.

Example: given `allow: docs/**` and `allow: docs/public-keys/*.pem`, the second
pattern is more specific and governs `docs/public-keys/ca.pem`.

Rationale: default-deny protects secrets, but repos must be able to exempt truly
public fixtures (e.g., `.env.example`, `docs/public-keys/*.pem`) without loss of
safety for adjacent files.

### 3.5 Bypass rules

Bypasses are temporary. They exist to enable incident response and one-off debugging
without requiring permanent `allow:` entries.

- A `bypass:` entry MUST have an `expires` date no more than **180 days** in the
  future. Implementations MUST refuse entries with later expiries.
- When a bypass is honored, implementations MUST emit an `info.acp-ignore-bypass-used`
  audit event including `approver`, `expires`, `path`, and `timestamp`.
- A bypass whose `expires` date has passed MUST be treated as `deny:`. Stale bypass
  entries SHOULD be removed, but expired entries MUST NOT silently grant access.

### 3.6 Audit requirements

An ACE L1 implementation MUST:

- Block every read attempt for a denied path BEFORE file content leaves the tool's
  process boundary
- Emit an `audit.log` entry for every deny-hit. Entry format:

  ```json
  {
    "timestamp": "2026-04-14T09:30:00Z",
    "severity": "violation",
    "rule": "acp-ignore-deny",
    "path": ".env.production",
    "tool": "host-tool-name/1.2.3",
    "actor": "opaque agent identifier",
    "matched_pattern": ".env.*"
  }
  ```

- Expose the audit log to the user on demand. Format SHOULD be SARIF 2.1.0 (§5.4).

An implementation MUST NOT claim L1 conformance while silently reading paths that it
then filters after the fact. Pre-read blocking is required.

### 3.7 Interaction with `.gitignore`

`.gitignore` governs version control. `.acp-ignore` governs AI agent read access.
A path MAY appear in one and not the other; they are independent.

Compliant tools MUST NOT assume that `.gitignore` implies `.acp-ignore`, and vice
versa. A `node_modules/` entry in `.gitignore` does not prevent an AI agent from
reading inside — only a matching `.acp-ignore` entry does.

---

## 4. Layer 2 — Context File Spec

### 4.1 Frontmatter schema

ACE context files (e.g., `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/*.mdc`,
`.github/instructions/*.instructions.md`) MAY begin with a YAML frontmatter block. When
present, the frontmatter MUST be valid YAML 1.2 and enclosed in `---` fences.

Field inventory:

| Field | Requirement | Type | Description |
|---|---|---|---|
| `spec-version` | REQUIRED | string | ACE spec version (e.g., `"0.1"`) |
| `applies-to` | REQUIRED | string \| string[] | Glob pattern(s) scoping this file |
| `activation` | REQUIRED | enum | One of `always` / `auto-attached` / `agent-requested` / `manual` (see §4.2) |
| `title` | RECOMMENDED | string | Short human-readable title |
| `description` | RECOMMENDED | string | ≤ 1024 chars; surfaces in agent UI |
| `status` | RECOMMENDED | enum | `active` / `draft` / `deprecated` (see §4.3) |
| `updated` | RECOMMENDED | string | ISO 8601 date of last meaningful edit |
| `deprecated-after` | OPTIONAL | string | ISO 8601 date; after which the file is stale |
| `language.comments` | OPTIONAL | string | BCP-47 tag, e.g., `ja` / `en` |
| `language.commits` | OPTIONAL | string | Same; for agent-authored commits |
| `output_language` | OPTIONAL | string | Agent response language preference |
| `tone` | OPTIONAL | string | Free text (e.g., "terse", "関西弁") |
| `spec-id` | OPTIONAL | string | Link to a Spec-Kit / RFC identifier |
| `excludeAgent` | OPTIONAL | string[] | Vendor extension (Copilot); excluded agent roles |
| `paths` | OPTIONAL | string[] | Vendor extension (Claude Skills) |

Unknown fields MUST NOT cause a hard error. Implementations SHOULD warn about
unrecognized fields to catch typos.

### 4.2 Activation modes

Four modes, chosen to subsume Cursor v2.2 and Copilot 2026-04 GA semantics:

| Mode | When the file's content is attached to the agent context |
|---|---|
| `always` | Every interaction, regardless of files being viewed |
| `auto-attached` | Automatically when a file matching `applies-to` is opened or edited |
| `agent-requested` | Only when the agent decides the file is relevant (via `description`) |
| `manual` | Only when the user explicitly references the file |

An implementation MUST implement at least `always` and `auto-attached`. `agent-requested`
and `manual` are RECOMMENDED.

### 4.3 Status lifecycle

```
draft → active → deprecated
```

- `draft`: work-in-progress; MAY be excluded from default activation by linters
- `active`: ready for agents to consume
- `deprecated`: superseded or scheduled for removal; host tools SHOULD warn when
  activating it

Linters SHOULD emit a `warn.deprecated-context-active` finding when a file with
`status: deprecated` is nonetheless activated for a session.

### 4.4 Stale detection

A context file is **stale** if any of:

- `status: deprecated`
- `deprecated-after` is in the past
- `updated` is older than 180 days AND the file uses `activation: always`

Stale files SHOULD be reported as `warn.stale-context`. Rationale: MITRE / NAACL
Findings 2024 showed that incorrect documentation degrades LLM performance more than
missing documentation does. Stale detection is therefore safety-critical, not cosmetic.

---

## 5. Lint Rules

### 5.1 Structural lint (REQUIRED for L2)

The following rules MUST be implemented:

| Rule ID | Severity | Description |
|---|---|---|
| `ace-fm-required-field` | error | Required frontmatter field missing |
| `ace-fm-invalid-enum` | error | `activation` / `status` outside allowed enum |
| `ace-fm-invalid-date` | error | `updated` / `deprecated-after` not ISO 8601 |
| `ace-fm-applies-to-empty` | error | `applies-to` resolves to no files |
| `ace-acp-ignore-malformed` | error | `.acp-ignore` syntax error |
| `ace-acp-ignore-bypass-no-approver` | error | `bypass:` missing `approver` |
| `ace-acp-ignore-bypass-no-expires` | error | `bypass:` missing `expires` |
| `ace-acp-ignore-bypass-too-long` | error | `bypass:` `expires` > 180 days out |
| `ace-acp-ignore-expired-bypass` | warn | `bypass:` past its `expires` date |
| `ace-stale-context` | warn | File is stale per §4.4 |
| `ace-deprecated-active` | warn | `status: deprecated` but activated |
| `ace-unicode-tag-found` | error | Invisible Unicode Tag char detected (§8.2) |
| `ace-bidi-override-found` | error | BiDi override char detected (§8.2) |
| `ace-zwsp-in-instruction` | warn | Zero-width space inside an instruction line |
| `ace-unknown-frontmatter-field` | info | Unrecognized (possibly misspelled) field |
| `ace-linkrot` | warn | Internal link to a missing file |

### 5.2 Semantic lint (OPTIONAL, draft)

Semantic lint uses an LLM to detect:

- Contradictions between multiple context files
- Ambiguous instructions ("sometimes", "maybe", "depending on…")
- Terminology drift (same concept named inconsistently)
- Dependencies on packages that do not exist

Semantic lint is OPTIONAL in v0.1. When implemented, it:

- MUST NOT run on every keystroke (cost)
- SHOULD run at PR time
- MUST assign a confidence score (0.0–1.0) to each finding
- MUST allow users to suppress findings below a configurable confidence threshold

Detailed ruleset and confidence calibration are out of scope for v0.1 and will be
stabilized in v0.2.

### 5.3 Severity model

| Severity | Semantics |
|---|---|
| `error` | Correctness or security violation; blocks CI by default |
| `warn` | Quality issue; surfaces in PR review, does not block by default |
| `info` | Observation; surfaces in verbose output only |

Implementations MUST allow per-rule severity override via configuration.

### 5.4 Output format (SARIF 2.1.0)

Linter output MUST be expressible as [SARIF 2.1.0]. Minimum required SARIF fields:

- `runs[].tool.driver.name` = ACE implementation name
- `runs[].tool.driver.semanticVersion` = implementation version
- `runs[].tool.driver.informationUri` = implementation URL
- `runs[].results[].ruleId` = ACE rule ID (e.g., `ace-fm-required-field`)
- `runs[].results[].level` = `error` / `warning` / `note`
- `runs[].results[].message.text`
- `runs[].results[].locations[].physicalLocation.artifactLocation.uri`
- `runs[].results[].locations[].physicalLocation.region.startLine`

Implementations MAY additionally emit human-friendly formats (pretty / JSON / GitHub
Actions annotations). SARIF is the normative interchange format.

[SARIF 2.1.0]: https://docs.oasis-open.org/sarif/sarif/v2.1.0/

---

## 6. Adapter Layer

Adapters read non-ACE-native context files and expose them through ACE's internal
representation. This lets repos adopt ACE validation without rewriting their existing
files.

### 6.1 `AGENTS.md`

- Repo root `AGENTS.md` MUST be read as `activation: always`, `applies-to: ["**/*"]`,
  `spec-version: "0.1"` unless a frontmatter block overrides.
- Nested `AGENTS.md` files (e.g., `packages/foo/AGENTS.md`) inherit `applies-to` scoped
  to their directory subtree.

### 6.2 `CLAUDE.md`

- Claude Code's `CLAUDE.md` MUST be read as `activation: always`, `applies-to: ["**/*"]`
  unless frontmatter overrides.
- `CLAUDE.md` files nested deeper scope to their subtree (mirror of `AGENTS.md`
  semantics).

### 6.3 Cursor `.cursor/rules/*.mdc`

- The three Cursor frontmatter fields map to ACE as follows:

| Cursor field | ACE field |
|---|---|
| `description` | `description` |
| `globs` | `applies-to` |
| `alwaysApply: true` | `activation: always` |
| `alwaysApply: false` + glob | `activation: auto-attached` |
| (description-only, no globs, no `alwaysApply`) | `activation: agent-requested` |

### 6.4 Copilot `.github/instructions/*.instructions.md`

- `applyTo` glob maps to `applies-to`.
- Personal > Repo > Org merge order is preserved; ACE treats each scope as a distinct
  context file with explicit precedence metadata on emit.

### 6.5 Windsurf rules (`.windsurfrules`)

- Activation modes (`always_on` / `manual` / `model_decision` / `glob`) map 1:1 to ACE
  modes (`always` / `manual` / `agent-requested` / `auto-attached`).
- The 6,000-char per-file and 12,000-char active-budget limits are advisory in ACE;
  lint MAY surface them as warnings but MUST NOT reject files that exceed them.

### 6.6 Adapter conformance

An adapter is ACE-conformant if, given any source file valid under the source format,
the adapter emits an ACE internal representation that the ACE linter can validate
without spurious errors.

---

## 7. Audit & Provenance

### 7.1 SARIF integration

L2 implementations MUST emit SARIF 2.1.0. This enables direct ingestion by:

- GitHub Code Scanning (Advanced Security)
- GitLab SAST report integration
- Azure DevOps advanced security
- Any generic SARIF viewer

### 7.2 Sigstore signing

Implementations SHOULD sign `.acp-ignore` and any context files intended as
source-of-truth using [Sigstore] (`cosign sign-blob`). Verification of signatures is
OPTIONAL in v0.1 and RECOMMENDED for enterprise deployments.

Signed files produce a `.sig` sibling plus a transparency log entry. Linters MAY
surface a `warn.unsigned-context` finding when operating in enterprise mode.

[Sigstore]: https://www.sigstore.dev/

### 7.3 SBOM generation

Implementations MAY generate an [SPDX] or [CycloneDX] SBOM listing all context files
and their hashes. This is OPTIONAL in v0.1 and becomes RECOMMENDED in v0.3 for
alignment with EU CRA 2027 requirements.

[SPDX]: https://spdx.dev/
[CycloneDX]: https://cyclonedx.org/

---

## 8. Security Considerations

### 8.1 Rules File Backdoor mitigation

Research (Pillar Security 2025, arXiv:2601.17548) has demonstrated that context files
can be exploited to inject hidden instructions into AI agents, with adaptive-attack
success rates exceeding 85%. Common vectors:

- Invisible Unicode (Zero-Width Space, U+200B; Unicode Tag characters U+E0000–U+E007F;
  BiDi override U+202E)
- Homoglyph substitution
- Instructions disguised as comments inside fenced code blocks

ACE mitigations (NORMATIVE in v0.1):

- `ace-unicode-tag-found` (error): Reject files containing any character in the range
  U+E0000–U+E007F outside of a `lang` tag block
- `ace-bidi-override-found` (error): Reject files containing U+202E, U+202D, U+2066,
  U+2067, U+2068
- `ace-zwsp-in-instruction` (warn): Flag U+200B / U+200C / U+200D / U+FEFF inside
  instruction lines (detection outside code blocks)

Linters MUST provide a `--render-invisible` / equivalent visualization mode that
displays these characters to the user when a finding is surfaced.

### 8.2 Supply-chain safety

- Context files imported from third-party sources (e.g., shared rule packs) SHOULD be
  pinned by hash.
- Linters SHOULD refuse to auto-apply context files that fail signature verification
  when operating in enterprise mode.

### 8.3 Agent-authored context

When an AI agent modifies a context file (e.g., a Claude Code session editing
`CLAUDE.md`), the change MUST be attributable. Implementations SHOULD:

- Require `Signed-off-by:` in the commit (per DCO)
- Record the agent identity (`tool-name/version` + model ID) in commit metadata
- Surface agent-authored changes distinctly in PR review

### 8.4 `.acp-ignore` itself

`.acp-ignore` is a trust anchor. Implementations MUST:

- Validate syntax before honoring any directive
- Reject the entire file on parse failure rather than partially honoring it
- Refuse to auto-edit `.acp-ignore` without explicit human approval

---

## 9. Versioning Policy

ACE follows [Semantic Versioning 2.0.0] at the spec level.

- **MAJOR** (1.0 → 2.0): Backwards-incompatible changes to the frontmatter schema,
  `.acp-ignore` syntax, or conformance levels
- **MINOR** (0.1 → 0.2): Backwards-compatible additions (new lint rules, new adapters,
  new optional frontmatter fields)
- **PATCH** (0.1.0 → 0.1.1): Editorial clarifications only

Draft versions (e.g., `0.1.0-draft`) MAY change incompatibly until they reach stable
status. Once declared stable, a minor version is immutable.

**Promotion from `draft` to stable** requires:

- Self-review and at least one external reviewer
- For stable `draft → implemented`: two independent implementations demonstrating
  interop (IETF convention)

[Semantic Versioning 2.0.0]: https://semver.org/spec/v2.0.0.html

---

## 10. Changelog

### v0.1.1 (2026-04-22)

Editorial patch. Resolves all five open issues from Appendix B:

- **§3.2** — Clarified `bypass:` block termination grammar (indented-continuation model
  with blank-line / new-directive as terminator; added BNF-equivalent prose + example).
- **§3.4** — Resolved `allow:` ordering ambiguity: most-specific pattern wins; last
  entry wins on tie.
- **Appendix B** — Items 3 (per-agent access control) and 4 (semantic lint score
  normalisation) explicitly deferred to v0.2 with rationale.
- **Added** `spec/v0.1/ja/protocol.md` — Japanese translation (informative).

### v0.1.0-draft (2026-04-14)

Initial public draft. No prior versions.

---

## 11. References

### Normative

- [RFC 2119] — Key words for use in RFCs to indicate requirement levels
- [RFC 8174] — Ambiguity of uppercase vs lowercase in RFC 2119 key words
- [SARIF 2.1.0] — OASIS Static Analysis Results Interchange Format
- [Semantic Versioning 2.0.0]

### Informative

- [AGENTS.md](https://agents.md) — AAIF cross-vendor root-file spec
- [MCP](https://modelcontextprotocol.io) — Model Context Protocol
- [AMP](https://github.com/Akari-OS/amp) — Agent Memory Protocol
- [M2C](https://github.com/Akari-OS/m2c) — Media to Context
- [Cursor Rules](https://docs.cursor.com/context/rules-for-ai)
- [Claude Code Skills](https://github.com/anthropics/claude-skills)
- [Copilot Instructions](https://docs.github.com/copilot/customizing-copilot/about-customizing-github-copilot-chat-responses)
- [Windsurf Rules](https://docs.windsurf.com/windsurf/cascade/memories)
- [Cline Memory Bank](https://docs.cline.bot/prompting/custom-instructions)
- [Vale](https://vale.sh/) — engine-less linter architecture
- [Ruff](https://docs.astral.sh/ruff/) — Rust-based linter performance reference
- [Sigstore](https://www.sigstore.dev/)
- [robots.txt](https://www.robotstxt.org/)
- [MITRE NAACL Findings 2024] — "Incorrect documentation harms more than missing documentation"
- [Pillar Security, Rules File Backdoor, 2025]
- [arXiv:2601.17548] — Adaptive attacks on agent context files

---

## 12. Appendix A — Minimal conforming example

A repository with the following files is a minimal L1-conforming ACE consumer:

```
my-repo/
├── .acp-ignore
├── AGENTS.md
└── docs/
    └── README.md
```

Where `.acp-ignore` contains:

```gitignore
# Default-deny list is shipped by the linter; only repo-specific
# entries need to be listed here.
deny: internal/**
allow: .env.example
```

And `AGENTS.md` begins with:

```yaml
---
spec-version: "0.1"
applies-to: "**/*"
activation: always
title: Repo conventions
status: active
updated: 2026-04-14
---
```

## 13. Appendix B — Remaining considerations for v0.2+

All five issues listed in v0.1.0-draft have been resolved or explicitly deferred in the
v0.1.1 editorial patch.

### Resolved in v0.1.1

| Issue | Resolution |
|---|---|
| `bypass:` block termination grammar | §3.2 now specifies indented-continuation model with prose + example |
| `allow:` ordering when multiple entries conflict | §3.4 now specifies most-specific-wins; last-in-file on tie |
| Japanese translation | `spec/v0.1/ja/protocol.md` added |

### Deferred to v0.2

- **Per-agent access control** — Expressing access control by agent identity
  (`@claude-code`, `@cursor`) rather than path requires inventing a capability language.
  Deferred to v0.2 to gather implementation experience first.
- **Semantic lint confidence score normalisation** — Normalising scores across LLM
  vendors requires sufficient benchmark data. Will be stabilised in v0.2 alongside the
  semantic lint ruleset.

Issues are tracked at <https://github.com/Akari-OS/ace/issues>.
