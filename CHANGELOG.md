# Changelog

All notable changes to OpenTopology are documented here.

## Unreleased

### Normative Changes

### Schema Changes

### Editorial Changes

### Governance Changes

## 0.2.0 - 2026-06-27

### Normative Changes

- Made chronology metadata first-class and breaking from otop-core 0.1.
- Required object `metadata.created_at` on every object.
- Replaced Evidence `produced_at` with `metadata.observed_at`.
- Removed `produced_at` from otop-core 0.2 Evidence.
- Added relationship chronology and event timestamp rules.
- Strengthened `depends_on` and `supersedes` cycle handling requirements.
- Added graph partition and external reference semantics.

### Schema Changes

- Added `schemas/otop-core-v0.2.schema.json`.
- Added mandatory object and Evidence metadata validation.
- Added UTC `Z` timestamp validation for chronology metadata.
- Rejected removed Evidence `produced_at`.

## 0.1.0 - 2026-06-25

### Added

- Initial OpenTopology Core draft specification.
- Initial JSON Schema for otop-core 0.1.
- Project governance, contribution, licensing, IPR, trademark, and security policies.
