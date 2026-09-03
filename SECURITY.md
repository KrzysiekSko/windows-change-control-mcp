# Security Policy

## Reporting a Vulnerability

Please report security vulnerabilities to Krzysztof Skomra via email at [security@rzeszow.pl](mailto:security@rzeszow.pl) or through the GitHub Security Advisory system.

We will acknowledge receipt of your report within 48 hours and provide a preliminary assessment.

We aim to provide an initial response within 72 hours, including a plan for addressing the issue if confirmed.

## Supported Versions

We provide security updates for the latest stable release branch.

## Disclosure Policy

We follow coordinated disclosure practices. Public disclosure will occur after a fix is available and users have had reasonable time to update.

## Security Invariants

This project enforces the following security invariants as architectural requirements:

- SEC-01: Deny by Default
- SEC-02: No Arbitrary Shell
- SEC-03: Read Is Not Write
- SEC-04: Plan Is Not Authorization
- SEC-05: No Self-Approval
- SEC-06: Rejected Write Means Zero Side Effect
- SEC-07: Authorization Must Bind to the Exact Plan
- SEC-08: Least Privilege
- SEC-09: Evidence Before Trust
- SEC-10: Supply Chain Is Part of the Trust Boundary

See [AGENTS.md](AGENTS.md) for full details.
