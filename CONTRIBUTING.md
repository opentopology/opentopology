# Contributing to OpenTopology

OpenTopology is developed as open documentation plus open implementation assets.
The goal is a specification that can be read, reused, implemented, tested, and
extended without permission from any single vendor.

## Contribution Types

Contributions may include:

- specification text
- schemas
- examples
- reference code
- tests and validation fixtures
- issue reports and design feedback
- governance and process improvements

## Licenses

By contributing, you agree that your contribution is licensed as follows:

- Specification prose: CC-BY-4.0, as described in `LICENSE-SPEC.md`.
- Schemas, examples, reference code, tests, and tooling: Apache-2.0, as
  described in `LICENSE`.

If a contribution touches both categories, each part is licensed according to
the file or asset type it modifies.

## Sign-Off

Every commit should include a Developer Certificate of Origin sign-off:

```text
Signed-off-by: Your Name <your.email@example.com>
```

Use:

```sh
git commit -s
```

The sign-off means you certify that you have the right to submit the
contribution under this repository's licenses and contribution policies.

## Normative Specification Changes

Changes to normative specification text include any change that affects what an
implementation MUST, SHOULD, or MAY do, including conformance language, schema
semantics, compatibility rules, extension rules, or required behavior.

Normative changes require:

- a pull request
- a clear problem statement
- implementation impact notes
- maintainer approval
- IPR review under `IPR.md`

Maintainers may request implementation experience, examples, tests, or a staged
release plan before accepting normative changes.

## Pull Request Expectations

Pull requests should:

- keep unrelated changes separate
- update schemas and examples when spec behavior changes
- update conformance or compatibility text when relevant
- include tests or validation fixtures for machine-readable behavior
- explain backward compatibility impact

## Issues and Proposals

Use issues for bug reports, ambiguity reports, design questions, and extension
requests. A larger change should start as an issue or proposal before a large
pull request.

## Code of Conduct

All participation is governed by `CODE_OF_CONDUCT.md`.
