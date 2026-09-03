# Windows Change Control MCP

Policy-enforced control plane for safe, auditable, and human-authorized Windows administration by AI agents.

## Repository
`windows-change-control-mcp`

## Project Positioning

Windows Change Control MCP is a policy-enforced control plane for safe, auditable, and human-authorized Windows administration by AI agents.

This project is **not** a generic PowerShell execution server and **not** a simple MCP wrapper around WinUtil.

Its primary architectural goal is to provide a governed layer between:

- AI models and agents,
- MCP runtimes,
- authorization and policy controls,
- privileged Windows operations,
- execution providers,
- human decision-makers,
- audit and assurance mechanisms.

The core product thesis is:

> AI should be allowed to propose and prepare administrative changes, but privileged execution must remain policy-controlled, explicitly authorized, observable, verifiable, and auditable.

The project treats WinUtil as the first execution provider, not as the product boundary.

## Getting Started

This repository is in the early stages of development. See the [Project Charter](docs/PROJECT-CHARTER.md) for more information.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Governance

See [GOVERNANCE.md](GOVERNANCE.md) for details on project governance.

## Security Policy

See [SECURITY.md](SECURITY.md) for details on security reporting and policies.

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for details on contributor conduct.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for details on contributing to this project.

