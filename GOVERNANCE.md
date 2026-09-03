# Governance Model

This project follows a governance-first approach to ensure that architectural decisions align with security, compliance, and executive objectives.

## Decision Making

Architectural decisions are made through Architecture Decision Records (ADRs) stored in `docs/decisions/`.

Major changes require an ADR and review by the project governance board.

## Roles

- **Project Architect**: Krzysztof Skomra - responsible for overall architectural direction and strategic positioning.
- **Maintainers**: Trusted contributors who review code and manage releases.
- **Contributors**: Anyone who contributes code, documentation, or other assets.

## Release Process

Releases are tagged and follow semantic versioning.

Each release must pass:
- Security tests (including rejection invariants)
- Evidence integrity tests
- Provider boundary validation
- Documentation updates

## Compliance

The project is designed to support compliance with frameworks such as ISO/IEC 27001, ISO/IEC 42001, NIST AI RMF, and GDPR principles, but does not guarantee compliance.

## Conflict Resolution

In case of disagreements, the project architect has the final say on architectural matters.

## Transparency

All governance documents, ADRs, and meeting minutes are stored in the repository.
