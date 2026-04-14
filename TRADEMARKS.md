# ACE — Trademark Policy

> Inspired by the [Node.js trademark policy](https://trademark-policy.openjsf.org/) and
> Kubernetes' Certified Kubernetes model. This document governs third-party use of the
> "ACE" and "Agent Context Engineering" names.

## Marks covered

This policy covers the following terms as used in relation to the ACE Framework:

- **ACE** (when used in the context of AI / agent / context-file tooling)
- **Agent Context Engineering**
- **Agent Context Engineering Framework**
- The ACE logo (once published; TBD)

Other uses of the word "ace" in general English (e.g., playing cards, pilots, tennis
scoring) are **not** covered.

## Ownership

The marks are currently held by the project's BDFL (Ryoma Nakajima / Akari-OS). On
transfer of the project to a neutral foundation — a condition for the v1.0 release — the
marks will be assigned to that foundation. Until then, the BDFL administers the policy
below on behalf of the community.

## Permitted uses (no written permission required)

You may use the name **ACE** to:

- Refer to the ACE Framework in documentation, blog posts, talks, books, and courses
- State that your tool "supports ACE", "reads `.acp-ignore`", or "is an ACE-compatible
  linter" **when it genuinely conforms to the published spec**
- Cite the spec in research papers and standards documents
- Use the name in source code comments and import paths (e.g., `package ace_linter`)

## Uses requiring permission

You must request permission (open an Issue on this repository) before:

- Naming a commercial product or service "ACE", "Agent Context Engineering Framework",
  or a confusingly similar variant
- Registering a domain that includes "ace" in combination with "agent", "context",
  "lint", or similar AI-agent terms, in a way that suggests official status
- Producing merchandise or swag bearing the name or logo

## Prohibited uses

You may **not**:

- Claim ACE compatibility for a tool that does not conform to the published spec
  (misleading marketing)
- Use the name or logo in a way that implies official endorsement, sponsorship, or
  affiliation with the Akari-OS organization, the ACE BDFL, or the future neutral
  foundation
- Modify the logo or create derivative marks that a reasonable observer could confuse
  with the official ACE mark
- Use the name in contexts that violate the [Code of Conduct](./CODE_OF_CONDUCT.md)

## Conformance claims

To state "ACE-conformant" or "ACE-compatible", your implementation must:

1. Respect `.acp-ignore` deny rules per the published spec
2. Produce SARIF 2.1.0 output when acting as a linter
3. Honor the frontmatter schema requirements for any context files it emits
4. Disclose its ACE version (`ACE v0.1`, etc.) in documentation

Certified-conformance testing (à la Certified Kubernetes) is out of scope for v0.1 and
may be introduced post-v1.0 by the neutral foundation.

## Contact

Questions or permission requests: open an Issue on this repository, or email
`info@kyo-kobo.com`.

## Revisions

This policy may be amended by the BDFL (and later, the foundation) via a standard PR +
RFC process. Material changes will be announced in [CHANGELOG.md](./CHANGELOG.md).
