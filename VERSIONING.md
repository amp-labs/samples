# Ampersand Manifest Schema Versioning

This document outlines the versioning strategy for the Ampersand manifest schema.

## Versioning Approach

The Ampersand manifest schema follows semantic versioning (`MAJOR.MINOR.PATCH`):

```yaml
specVersion: 1.0.0
```

- **MAJOR**: Incremented for breaking changes that require manifest updates
- **MINOR**: Incremented for new features added in a backward-compatible manner
- **PATCH**: Incremented for backward-compatible bug fixes and clarifications

## Version Specification

The version is specified in the manifest file with the `specVersion` field:

```yaml
specVersion: 1.0.0
integrations:
  # ...
```

## Compatibility Guarantees

- **Backward compatibility** is guaranteed within the same MAJOR version
- Each MAJOR version is supported for at least 12 months after a new MAJOR version is released
- Manifests for older MAJOR versions will continue to work with that version of the schema

## Current Version

| Version | Status | Released | Support End |
|---------|--------|----------|-------------|
| 1.0.0   | Current | 2023 | TBD |

## Breaking Changes Policy

1. Breaking changes only occur in MAJOR version increments
2. Breaking changes are announced at least 3 months before release
3. Migration guides are provided between MAJOR versions
4. Deprecation notices are included in documentation before features are removed

## Migration Between Versions

When a new MAJOR version is released, a migration guide will be provided that explains:

- What changes were made and why
- How to update your manifests to the new version
- Any automated migration tools that are available

## Version Detection

Ampersand tools automatically detect the manifest version from the `specVersion` field and validate against the appropriate schema version.

## Schema Location

Each version of the schema has its own JSON Schema definition at `/schema/vX/ampersand-schema.json`.
