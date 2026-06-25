# OpenTopology Core Specification

**Short name:** otop-core  
**Version:** 0.1  
**Status:** Draft  
**Audience:** Tool authors, compiler implementers, repository indexers, AI-agent platform builders, governance tooling vendors

## Normative Language

The terms `MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY` are normative when they appear in uppercase, as described by RFC 2119 and RFC 8174. Lowercase uses of `must`, `should`, and `may` are also normative when they appear in conformance requirements or field definitions.

## 1. Introduction

OpenTopology, abbreviated as **otop**, is an open graph standard for representing relationships between software intent, constraints, policies, implementation artifacts, and verification evidence.

The purpose of otop is to provide a portable topology layer that allows independent tools to exchange structured knowledge about what software is intended to do, where that intent is implemented, what governs it, and what evidence supports it.

OpenTopology defines an application and intent topology layer. It is not a network topology, service mesh topology, VPC topology, deployment map, or infrastructure inventory standard, although those systems may be represented as Artifacts or referenced by extension profiles.

An otop graph allows tools to answer questions such as:

- What Intent did this Constraint come from?
- Which Policies govern this Constraint?
- Which Artifacts implement this Constraint?
- Which Evidence verifies this Constraint?
- Which Constraints lack implementation?
- Which Evidence has become stale?
- Which Constraints conflict?

## 2. Goals

The goals of otop-core are to define:

- A shared graph model.
- Core object kinds.
- Required object fields.
- Core relationship types.
- Stable identity rules.
- Artifact locator semantics.
- Evidence semantics.
- Extension rules.
- Basic conformance requirements.

## 3. Non-Goals

otop-core does not define:

- A specific database.
- A specific compiler.
- A specific parser.
- A specific AI-agent runtime.
- A specific transport protocol.
- A specific policy engine.
- A specific test framework.
- A specific requirements authoring workflow.
- A guarantee that all semantic conflicts can be detected automatically.

otop may be used by AI-agent systems, but AI-agent behavior is not part of otop-core.

## 4. Terminology

An **otop graph** is a directed labeled property graph.

A graph consists of:

- **Objects**, represented as typed graph nodes.
- **Relationships**, represented as typed graph edges.
- **Properties**, represented as structured metadata on objects and relationships.

The five core object kinds are:

- **Intent**
- **Constraint**
- **Policy**
- **Artifact**
- **Evidence**

The term "node" may be used when describing graph mechanics, but public object kinds should use the domain terms above.

## 5. Core Object Kinds

Every object must include `id`, `kind`, and `title`. Implementations may add optional fields and namespaced extension fields as described in the extension model.

Objects may include `lifecycle`. When present, `lifecycle` should use one of the core values defined in Section 5.6 or a namespaced extension value.

### 5.1 Intent

An **Intent** represents human-authored or tool-authored desired software behavior, system shape, product outcome, architecture decision, or requirement source.

Examples include product requirements, user stories, specification sections, architecture decisions, acceptance criteria, threat-model items, and customer outcomes.

Required fields:

```yaml
id: intent.auth.session-validation
kind: intent
title: User session validation
```

Recommended fields:

```yaml
intent_type: requirement
source:
  kind: markdown
  path: specs/auth/session-validation.md
lifecycle: active
owner: platform-security
```

### 5.2 Constraint

A **Constraint** represents a specific obligation, invariant, rule, or acceptance criterion that can be checked, reviewed, implemented, or verified.

Examples include:

- `session.token.entropy >= 256`
- `api.timeout <= 200ms`
- `user.password.length >= 12`
- `all customer data deletion requests must be completed within 30 days`

Required fields:

```yaml
id: constraint.auth.session-token-entropy
kind: constraint
title: Session tokens must provide sufficient entropy
```

Recommended fields:

```yaml
modality: MUST
constraint_type: security
checkability: dynamic
assertion: session.token.entropy >= 256
expression_language: cel
severity: high
lifecycle: active
```

### 5.3 Policy

A **Policy** represents a governing source, rule set, standard, control, architectural rule, compliance requirement, or organizational guardrail.

A Policy usually governs one or more Constraints.

Examples include ISO 27001 cryptographic controls, GDPR data deletion policy, internal secure coding standards, approved platform architecture, engineering golden paths, and data retention policy.

Required fields:

```yaml
id: policy.security.iso27001.crypto-controls
kind: policy
title: Cryptographic controls
```

Recommended fields:

```yaml
policy_type: security_control
authority: external
reference: ISO27001-A.10
lifecycle: active
```

### 5.4 Artifact

An **Artifact** represents a concrete implementation or project artifact.

Artifacts may be source code, tests, configuration, infrastructure, generated files, runtime endpoints, or deployment resources.

Examples include source files, functions, classes, API routes, test cases, configuration keys, Terraform resources, Kubernetes manifests, database migrations, and container images.

Required fields:

```yaml
id: artifact.auth.validate-session
kind: artifact
title: validateSession
artifact_type: source_symbol
```

Recommended fields:

```yaml
language: typescript
locator:
  repository: github.com/example/app
  path: src/auth/session.ts
  symbol: validateSession
  range:
    start_line: 42
    end_line: 87
  digest: sha256:8f3c3a92b3
```

AST data may be used as part of an Artifact locator, but an Artifact is not itself required to be an AST node.

### 5.5 Evidence

**Evidence** represents proof, observation, result, attestation, waiver, or review record related to a Constraint, Policy, or Artifact.

Examples include unit test results, integration test results, static-analysis results, policy scans, runtime observations, manual reviews, security approvals, waivers, and external audit records.

Required fields:

```yaml
id: evidence.auth.session-token-entropy.test.2026-06-20
kind: evidence
title: Session token entropy test result
evidence_type: unit_test_result
result: pass
```

Recommended fields:

```yaml
produced_at: 2026-06-20T12:00:00Z
tool:
  name: vitest
  version: 2.1.0
confidence: deterministic
digest: sha256:44ac91
subjects:
  - id: constraint.auth.session-token-entropy
    kind: constraint
    digest: sha256:7c91f0
    version: 3
  - id: artifact.auth.validate-session
    kind: artifact
    digest: sha256:8f3c3a92b3
    commit: 8f3c3a92b3
provenance:
  actor:
    id: user.alice
    type: human
  producer:
    name: vitest
    version: 2.1.0
  source:
    kind: ci
    run_id: 123456
signature:
  algorithm: ed25519
  key_id: user.alice.ssh
  payload_digest: sha256:44ac91
  signature: base64:MEUCIQD...
```

Valid evidence results are:

```text
pass
fail
unknown
waived
not_applicable
```

`subjects` records the object or relationship states the Evidence is about. Each subject should include the target object or relationship `id` and may include `kind`, `relationship_type`, `digest`, `commit`, and `version`.

`provenance` records who or what produced the Evidence. It may describe a human actor, an automated producer, a source system, or all three.

`signature` records an optional cryptographic binding over the Evidence payload. otop-core defines the field shape but does not require a specific signing ecosystem, algorithm, key management system, or attestation framework.

When `signature` is present, `payload_digest` is required. otop-core 0.1 does not define canonical serialization for signed payloads. Profiles may define canonicalization, signing, and verification requirements.

Profiles that define signed Evidence should use an existing deterministic serialization where possible. Recommended baselines include RFC 8785 JSON Canonicalization Scheme (JCS) for JSON payloads, or an explicitly specified deterministic YAML serialization. Profiles should avoid signing presentation details that are likely to change during semantically equivalent parse and re-emit operations.

### 5.6 Lifecycle

Objects may include `lifecycle` to describe their lifecycle state.

Core lifecycle values are:

```text
draft
active
deprecated
superseded
archived
```

Implementations should treat unknown unnamespaced lifecycle values as invalid. Custom lifecycle values should be namespaced through the extension model.

## 6. Core Relationship Types

Relationships connect objects in an otop graph.

Each relationship must include `from`, `to`, and `type`.

Relationships may include:

- `id`: a stable relationship identifier.
- `metadata`: lightweight non-authoritative relationship metadata.
- `extensions`: namespaced extension data.

Relationship `metadata` may include fields such as `confidence`, `created_at`, `created_by`, `inferred_by`, `method`, and `note`. Metadata is suitable for automation hints and review context. Authoritative proof, waivers, attestations, and conflict resolutions should be represented as Evidence.

Example:

```yaml
id: rel.constraint.auth.session-token-entropy.derived-from.intent.auth.session-validation
from: constraint.auth.session-token-entropy
to: intent.auth.session-validation
type: derived_from
metadata:
  confidence: high
  created_at: 2026-06-20T12:00:00Z
  created_by: user.alice
  method: human_review
```

### 6.1 derived_from

Connects a Constraint to the Intent, Policy, or source from which it was derived.

```text
Constraint --derived_from--> Intent
```

### 6.2 governed_by

Connects an Intent, Constraint, or Artifact to a Policy.

```text
Constraint --governed_by--> Policy
```

### 6.3 implemented_by

Connects an Intent or Constraint to an Artifact that implements it.

```text
Constraint --implemented_by--> Artifact
```

### 6.4 verified_by

Connects a Constraint, Policy, or Artifact to Evidence.

```text
Constraint --verified_by--> Evidence
```

The `verified_by` relationship identifies that Evidence supports the source object and provides an operational shortcut for graph traversal. Cryptographic state binding and deterministic staleness should be recorded in the Evidence `subjects` field. When `verified_by` and Evidence `subjects` disagree, `subjects` is the source of truth for cryptographic binding.

### 6.5 depends_on

Connects one object to another object required for interpretation, operation, or correctness.

```text
Constraint --depends_on--> Constraint
```

Tools that traverse `depends_on` relationships must implement cycle detection. Circular operational dependencies should be rejected as malformed graphs.

### 6.6 conflicts_with

Declares that two objects are incompatible.

```text
Constraint --conflicts_with--> Constraint
```

Current unresolved conflicts should remain represented as `conflicts_with` relationships. Accepted risk, override, mute, and resolution decisions should be represented as Evidence that targets the relationship `id`, preserving historical context without relying on edge deletion.

### 6.7 supersedes

Declares that one object replaces another.

```text
Constraint --supersedes--> Constraint
```

Tools that traverse `supersedes` relationships must implement cycle detection. `supersedes` lineage must be acyclic; circular lineage should be rejected as a malformed graph.

### 6.8 waived_by

Connects a Constraint or Policy to Evidence that records an approved exception.

```text
Constraint --waived_by--> Evidence
```

## 7. Canonical Topology

The recommended canonical topology is:

```text
Constraint --derived_from--> Intent
Constraint --governed_by--> Policy
Constraint --implemented_by--> Artifact
Constraint --verified_by--> Evidence
```

This makes Constraint the central query point.

Given a Constraint, a tool should be able to answer:

- Where did this Constraint come from?
- What governs it?
- Where is it implemented?
- How is it verified?

## 8. Manifest Format

An otop manifest may be represented as YAML or JSON.

A manifest contains:

- otop version.
- Graph metadata.
- Objects.
- Relationships.
- Optional extension data.
- Optional signatures.

### 8.1 Schema Binding

Manifests should declare their schema when the file format supports it.

JSON manifests should use the standard `$schema` key:

```json
{
  "$schema": "https://opentopology.dev/schemas/otop-core-0.1.schema.json",
  "otop_version": 0.1
}
```

YAML manifests should use an IDE-compatible schema directive comment:

```yaml
# yaml-language-server: $schema=https://opentopology.dev/schemas/otop-core-0.1.schema.json
otop_version: 0.1
```

Schema binding allows editors, language servers, and general-purpose validators to provide validation without requiring a custom tool.

Versioned schema URLs are immutable once published. `otop_version: 0.1` binds a manifest to the `0.1` schema family. Compatible corrections may be published with explicit patch URLs, such as `otop-core-0.1.1.schema.json`. Moving aliases may exist for convenience, but examples should use immutable versioned URLs.

Minimal valid manifest:

```yaml
# yaml-language-server: $schema=https://opentopology.dev/schemas/otop-core-0.1.schema.json
otop_version: 0.1

graph:
  id: graph.example.minimal
  title: Minimal Example Topology

objects:
  - id: intent.example.minimal
    kind: intent
    title: Minimal intent

relationships: []
```

Example:

```yaml
# yaml-language-server: $schema=https://opentopology.dev/schemas/otop-core-0.1.schema.json
otop_version: 0.1

graph:
  id: graph.example.auth-service
  title: Example Auth Service Topology
  repository: github.com/example/auth-service
  commit: 8f3c3a92b3

objects:
  - id: intent.auth.session-validation
    kind: intent
    intent_type: requirement
    title: User session validation
    lifecycle: active
    source:
      kind: markdown
      path: specs/auth/session-validation.md

  - id: constraint.auth.session-token-entropy
    kind: constraint
    constraint_type: security
    title: Session tokens must provide sufficient entropy
    lifecycle: active
    modality: MUST
    checkability: dynamic
    assertion: session.token.entropy >= 256
    expression_language: cel

  - id: policy.security.iso27001.crypto-controls
    kind: policy
    policy_type: security_control
    title: Cryptographic controls
    lifecycle: active
    authority: external
    reference: ISO27001-A.10

  - id: artifact.auth.validate-session
    kind: artifact
    artifact_type: source_symbol
    title: validateSession
    lifecycle: active
    language: typescript
    locator:
      path: src/auth/session.ts
      symbol: validateSession
      range:
        start_line: 42
        end_line: 87
      digest: sha256:8f3c3a92b3

  - id: evidence.auth.session-token-entropy.test.2026-06-20
    kind: evidence
    evidence_type: unit_test_result
    title: Session token entropy test
    result: pass
    produced_at: 2026-06-20T12:00:00Z
    tool:
      name: vitest
      version: 2.1.0
    confidence: deterministic
    digest: sha256:44ac91
    subjects:
      - id: constraint.auth.session-token-entropy
        kind: constraint
        digest: sha256:7c91f0
        version: 3
      - id: artifact.auth.validate-session
        kind: artifact
        digest: sha256:8f3c3a92b3
        commit: 8f3c3a92b3
      - id: rel.constraint.auth.session-token-entropy.implemented-by.artifact.auth.validate-session
        relationship_type: implemented_by
        digest: sha256:90b1e2
    provenance:
      actor:
        id: ci.github-actions
        type: automation
      producer:
        name: vitest
        version: 2.1.0
      source:
        kind: ci
        run_id: 123456
    signature:
      algorithm: ed25519
      key_id: ci.github-actions.2026-06
      payload_digest: sha256:44ac91
      signature: base64:MEUCIQD...

relationships:
  - id: rel.constraint.auth.session-token-entropy.derived-from.intent.auth.session-validation
    from: constraint.auth.session-token-entropy
    to: intent.auth.session-validation
    type: derived_from
    metadata:
      confidence: high
      created_at: 2026-06-20T12:00:00Z
      created_by: user.alice
      method: human_review

  - id: rel.constraint.auth.session-token-entropy.governed-by.policy.security.iso27001.crypto-controls
    from: constraint.auth.session-token-entropy
    to: policy.security.iso27001.crypto-controls
    type: governed_by

  - id: rel.constraint.auth.session-token-entropy.implemented-by.artifact.auth.validate-session
    from: constraint.auth.session-token-entropy
    to: artifact.auth.validate-session
    type: implemented_by
    metadata:
      confidence: medium
      inferred_by:
        tool: example-indexer
        version: 0.4.0
      method: static_analysis
      note: Symbol reference inferred from call graph.
    extensions:
      com.example.indexer:
        rule_id: callgraph.symbol-match

  - id: rel.constraint.auth.session-token-entropy.verified-by.evidence.auth.session-token-entropy.test.2026-06-20
    from: constraint.auth.session-token-entropy
    to: evidence.auth.session-token-entropy.test.2026-06-20
    type: verified_by
```

## 9. Identity Rules

Every object in an otop graph must have a stable `id`.

Object IDs should be:

- Unique within a graph.
- Stable across graph updates.
- Human-readable where possible.
- Namespaced to avoid collisions.
- Independent of line numbers.
- Suitable for diffing and review.

Relationship IDs are optional. When present, a relationship ID must be unique within the graph.

Relationship IDs should be:

- Stable while the relationship's `from`, `to`, and `type` semantics remain stable.
- Human-readable where possible.
- Namespaced to avoid collisions.
- Suitable for Evidence `subjects`.
- Suitable for diffing and review.

Recommended ID style:

```text
intent.auth.session-validation
constraint.auth.session-token-entropy
policy.security.iso27001.crypto-controls
artifact.auth.validate-session
evidence.auth.session-token-entropy.test.2026-06-20
rel.constraint.auth.session-token-entropy.implemented-by.artifact.auth.validate-session
```

## 10. Artifact Locators

Artifact locators describe where an Artifact can be found.

A locator may include:

- repository
- commit
- path
- symbol
- range
- digest
- language
- parser metadata
- runtime endpoint
- package identifier
- image reference

If `repository` is defined or can be inferred from graph metadata, `path` values must be interpreted as relative to the root of that repository. Tools should not interpret locator paths relative to the manifest file location or an arbitrary working directory unless an extension profile explicitly defines that behavior.

Line ranges are hints, not stable identity. AST data may be included as optional locator metadata.

Example:

```yaml
locator:
  path: src/auth/session.ts
  symbol: validateSession
  range:
    start_line: 42
    end_line: 87
  ast:
    parser: tree-sitter-typescript
    node_type: function_declaration
  digest: sha256:8f3c3a92b3
```

## 11. Evidence Semantics

Verification should be represented as Evidence, not only as mutable status.

A Constraint may be considered verified when sufficient valid Evidence objects are connected through `verified_by` relationships.

Evidence may become stale if:

- The related Constraint changes.
- The related Artifact changes.
- The related Policy changes.
- The verification method changes.
- The Evidence expires.
- A dependency changes.

For deterministic staleness computation, Evidence should record `subjects` snapshots for the object and relationship states it verifies. A tool should compare each subject snapshot against the current graph, object, relationship, Artifact locator, commit, version, or digest state available to that tool. If a subject no longer matches the current state, the Evidence should be treated as stale for that subject.

Tools may compute verification summaries, but summaries should be derived from Evidence objects and their `subjects`. The underlying Evidence object should remain immutable historical data.

## 12. Conflict Semantics

otop supports explicit representation of conflicts.

Conflict types may include:

```text
syntactic
structural
logical
policy
semantic
```

Tools must distinguish deterministic conflicts from inferred conflicts.

A tool must not claim deterministic semantic conflict detection unless the conflict is backed by a formal rule, declared contradiction, or approved human review.

Conflict resolution is historical graph data. Tools should not rely on deleting a `conflicts_with` relationship as the only representation of resolution. A conflict may be accepted, muted, overridden, or resolved by creating Evidence whose `subjects` include the `conflicts_with` relationship `id`. That Evidence may be connected with `waived_by` when it records an accepted exception, or with `verified_by` when it records a verified resolution.

## 13. Extension Model

Implementations may define custom fields, object subtypes, relationship types, and profiles.

Extensions must:

- Preserve core semantics.
- Avoid redefining core object kinds.
- Use namespaced identifiers.
- Be safely ignored by tools that do not understand them.
- Preserve unknown fields where possible.

Example:

```yaml
extensions:
  com.example.risk:
    risk_score: 8.7
    threat_model: STRIDE
```

## 14. Conformance

A tool conforms to `otop-core` if it can:

- Parse a valid otop manifest.
- Parse schema-bound JSON and YAML manifests.
- Validate required object fields.
- Validate required relationship fields.
- Resolve object IDs.
- Resolve relationship IDs when present.
- Validate core lifecycle values when `lifecycle` is present.
- Preserve unknown extension fields where possible.
- Preserve unknown relationship extension fields where possible.
- Export a valid otop graph.
- Reject invalid core object kinds.
- Reject relationships that reference missing objects.
- Reject Evidence subjects that reference missing objects or missing relationships when the referenced `id` is expected to resolve in the graph.
- Detect cycles when traversing `depends_on` and `supersedes`.
- Respect immutable versioned schema URLs when it uses remote schemas for validation.

A tool that claims support for signed Evidence must also parse the standard `provenance` and `signature` fields, require `payload_digest` when `signature` is present, and report whether the signature is valid, invalid, unverifiable, or absent. If the tool does not implement a profile-defined canonicalization rule for the signed payload, it should report the signature as unverifiable rather than valid. Profiles that define canonicalization should prefer established deterministic serialization rules such as RFC 8785 JCS for JSON.

Additional profiles may define stronger requirements.

Examples include:

- `otop-trace`
- `otop-verify`
- `otop-governance`
- `otop-delta`
- `otop-agent`

## 15. Security Considerations

otop graphs may influence software generation, verification, compliance workflows, and agent behavior.

Implementations should consider:

- Forged Evidence.
- Stale graph replay.
- Unauthorized Policy changes.
- Malicious Constraint weakening.
- Prompt injection through text fields.
- Context poisoning.
- Waiver abuse.
- Signature confusion.
- Tool impersonation.

Implementations should support:

- Graph digests.
- Evidence digests.
- Actor identity.
- Signed Evidence.
- Signed graph states.
- Audit logs.
- Authorization controls for waivers and verification status.

## 16. Relationship to Implementations

otop-core is implementation-neutral.

A tool may implement otop using:

- A local file.
- A database.
- A graph engine.
- A CLI.
- A CI plugin.
- An IDE extension.
- An MCP server.
- A hosted API.

No implementation is required by otop-core.

## 17. References

### 17.1 Normative References

- RFC 2119: Key words for use in RFCs to indicate requirement levels.
- RFC 8174: Ambiguity clarification for uppercase and lowercase requirement keywords.
- RFC 3986: Uniform Resource Identifier (URI) syntax for schema URLs and locator references.
- RFC 3339: Date and time format used by timestamp fields such as `produced_at`.
- RFC 4648: Base64 encoding used by signature examples.

### 17.2 Informative References

- RFC 8785: JSON Canonicalization Scheme (JCS), recommended for profiles that define signed JSON payloads.
- RFC 8032: Ed25519 and related Edwards-curve signature algorithms, referenced by signature examples.
- JSON Schema: External schema-validation vocabulary for otop JSON and YAML-compatible schema artifacts. The schema artifact defines the specific JSON Schema draft it uses.

## 18. Core Registries

| Registry | Values |
| --- | --- |
| Object kinds | `intent`, `constraint`, `policy`, `artifact`, `evidence` |
| Relationship types | `derived_from`, `governed_by`, `implemented_by`, `verified_by`, `depends_on`, `conflicts_with`, `supersedes`, `waived_by` |
| Evidence results | `pass`, `fail`, `unknown`, `waived`, `not_applicable` |
| Lifecycle values | `draft`, `active`, `deprecated`, `superseded`, `archived` |
| Conflict types | `syntactic`, `structural`, `logical`, `policy`, `semantic` |

## 19. Summary

OpenTopology defines a portable software topology graph.

Its five core object kinds are:

```text
Intent
Constraint
Policy
Artifact
Evidence
```

Its canonical relationship pattern is:

```text
Constraint --derived_from--> Intent
Constraint --governed_by--> Policy
Constraint --implemented_by--> Artifact
Constraint --verified_by--> Evidence
```

This allows independent tools to connect software intent, governing policy, implementation artifacts, and verification evidence in a common format.