# OpenTopology Governance

OpenTopology starts with lightweight governance and is intended to remain
foundation-ready if adoption grows.

## Stewardship

OpenTopology is currently stewarded by the initial maintainers listed below.
Maintainers are responsible for repository administration, release management,
review, and final approval of normative changes.

Initial maintainers:

- Stuart Preston (stuart@pendrica.com)

Additional maintainers may be added by maintainer consensus.

## Decision Making

Routine editorial changes, examples, typo fixes, and non-normative
clarifications may be approved by one maintainer.

Normative specification changes require maintainer consensus. Consensus means
there is broad agreement, no unresolved sustained objection from a maintainer,
and documented consideration of implementer impact.

If consensus cannot be reached, maintainers should:

1. identify the disputed requirement
2. document each position in the issue or pull request
3. request implementation or user evidence where possible
4. defer the change, narrow its scope, or move it to an extension proposal

## Normative Change Control

A normative change is any change that affects required, recommended, or
permitted implementation behavior.

Normative changes should include:

- rationale
- compatibility impact
- schema impact, if any
- example updates, if useful
- IPR considerations under `IPR.md`

## Release Stages

OpenTopology versions use these stages:

- Draft: active design; breaking changes are allowed.
- Candidate: feature shape is close to stable; breaking changes require clear
  justification.
- Stable: implementers can rely on compatibility rules.
- Deprecated: still documented, but no longer recommended for new adoption.
- Superseded: replaced by a newer version.

## Versioning

Before 1.0, minor versions may include breaking changes. Each draft should make
breaking changes visible in release notes or the relevant specification
document.

At 1.0 and later:

- patch releases should not introduce breaking normative changes
- minor releases may add backward-compatible features
- major releases may introduce breaking changes

## Compatibility and Conformance

Compatibility claims must name the implemented OpenTopology version and any
implemented profile or extension.

A conformant implementation must satisfy the conformance requirements in the
relevant specification and validate required machine-readable assets against
the applicable schemas.

The maintainers may reject or challenge misleading compatibility claims under
`TRADEMARK.md`.

## Extensions

Extensions should use namespaced identifiers and avoid redefining core
semantics. Widely used extensions may be proposed for registration or future
standardization.

Extension registration rules may be added as adoption grows.

## Schema Publication

Published schema URLs should be treated as immutable once released as stable.
Draft schema URLs may change, but changes should be visible in git history and
release notes.

## Appeals

If a contributor believes a decision was made incorrectly, they may open an
issue marked `governance` that explains the concern and proposed resolution.
Maintainers should respond with the decision rationale and any next step.
