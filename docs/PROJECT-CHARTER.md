# Project Charter

## Project Name
Windows Change Control MCP

## Vision
To provide a governed layer between AI agents and privileged Windows operations, ensuring that AI-initiated changes are policy-controlled, explicitly authorized, observable, verifiable, and auditable.

## Mission
To build a policy-enforced control plane that allows AI agents to propose and prepare administrative changes on Windows systems while maintaining strict separation between intent and execution, with human oversight and comprehensive evidence generation.

## Goals
1. Separate AI change intent from execution authority
2. Provide policy constraints on what agents may attempt
3. Enable human or independent control domain authorization for sensitive changes
4. Bind authorization to exact operations
5. Prove rejected operations have zero side effects
6. Allow provider swapping without MCP contract changes
7. Isolate privileged execution from the AI-facing control plane
8. Generate trustworthy evidence for administrative actions
9. Keep AI-assisted operations accountable to enterprise governance
10. Establish WinUtil as the first execution provider, not the product boundary

## Scope
In Scope:
- System inventory and capability discovery
- Change planning and risk classification
- Policy evaluation and approval workflows
- Controlled application installation
- Selected reversible Windows configuration changes
- Provider abstraction
- Post-change verification
- Evidence collection and rollback metadata
- Execution provenance

Out of Scope (by default):
- Unrestricted PowerShell or arbitrary script execution
- Arbitrary registry or service manipulation
- Unrestricted Windows Defender or firewall changes
- Boot configuration changes
- Destructive debloat operations
- Arbitrary Windows Update policy changes
- Privilege escalation helpers
- Credential harvesting
- Security-control bypass
- Remote arbitrary execution
- Hidden persistence mechanisms

## Success Criteria
The project succeeds if it demonstrates that:
1. AI agents can interact with Windows without unrestricted privileged shell access
2. Change intent is separated from execution authority
3. Policies constrain agent actions
4. Humans or independent control domains can authorize sensitive changes
5. Authorization is bound to exact operations
6. Rejected operations have zero side effects
7. Providers can be changed without MCP contract redesign
8. Privileged execution is isolated from the AI-facing control plane
9. Administrative actions produce trustworthy evidence
10. AI-assisted operations remain accountable to enterprise governance

## Roles
- Project Architect: Krzysztof Skomra
- Maintainers: To be appointed
- Contributors: Open

## Timeline
See the development roadmap in AGENTS.md for phased implementation gates.

## License
Apache License 2.0
