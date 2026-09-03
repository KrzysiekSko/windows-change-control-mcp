# MCP v0.1 Contract — Gate 2C

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b`
Status: **DESIGN ONLY — no implementation exists.**

> This document defines the MCP tool contracts for the read-only v0.1 release.
> It specifies the public API surface that the MCP server will expose to
> AI agents, built on top of the canonical operations defined in Gate 2A
> and the provider boundary defined in Gate 2B.

## Public MCP Tools — v0.1

```
system_inventory
application_inventory
tweak_state
provider_capabilities
update_status
system_configuration_summary
```

Each tool maps 1:1 to a canonical operation.

---

## Tool: system_inventory

```json
{
  "name": "system_inventory",
  "description": "Returns structured information about the current Windows system: OS version, hostname, edition, install date, last boot time, secure boot, and architecture. This is a read-only observation with no system state mutation.",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "input_schema": {
    "type": "object",
    "properties": {},
    "additionalProperties": false
  },
  "output_schema": {
    "type": "object",
    "required": ["operation_id", "timestamp", "hostname", "os_name", "os_version", "os_build", "architecture", "provider"],
    "properties": {
      "operation_id": { "type": "string", "const": "OP-001" },
      "timestamp": { "type": "string", "format": "date-time" },
      "hostname": { "type": "string" },
      "os_name": { "type": "string" },
      "os_version": { "type": "string" },
      "os_build": { "type": "string" },
      "edition": { "type": "string" },
      "install_date": { "type": ["string", "null"], "format": "date-time" },
      "last_boot_time": { "type": ["string", "null"], "format": "date-time" },
      "secure_boot": { "type": ["boolean", "null"] },
      "bitlocker_status": { "type": ["string", "null"] },
      "architecture": { "type": "string", "enum": ["x64", "ARM64"] },
      "provider": { "type": "string" },
      "provider_version": { "type": ["string", "null"] }
    }
  },
  "error_schema": {
    "type": "object",
    "properties": {
      "error_code": { "type": "string", "enum": ["INVALID_REQUEST", "PROVIDER_UNAVAILABLE", "OBSERVATION_FAILED", "EVIDENCE_GENERATION_FAILED"] },
      "message": { "type": "string" },
      "operation_id": { "type": "string", "const": "OP-001" },
      "timestamp": { "type": "string", "format": "date-time" }
    }
  }
}
```

---

## Tool: application_inventory

```json
{
  "name": "application_inventory",
  "description": "Returns installed applications across Win32, AppX, winget, and Chocolatey sources. User-scoped only in v0.1 — does not query machine-wide AppX packages. May touch network cache for winget/choco list operations. Read-only, no install/upgrade/remove.",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "input_schema": {
    "type": "object",
    "properties": {
      "filters": {
        "type": "object",
        "properties": {
          "source": {
            "type": "array",
            "items": { "type": "string", "enum": ["win32", "appx", "winget", "choco"] }
          },
          "name_pattern": { "type": "string", "description": "Simple substring filter on application name" }
        },
        "additionalProperties": false
      }
    },
    "additionalProperties": false
  },
  "output_schema": {
    "type": "object",
    "required": ["operation_id", "timestamp", "applications", "total", "provider"],
    "properties": {
      "operation_id": { "type": "string", "const": "OP-002" },
      "timestamp": { "type": "string", "format": "date-time" },
      "applications": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["name", "source", "scope"],
          "properties": {
            "name": { "type": "string" },
            "version": { "type": ["string", "null"] },
            "source": { "type": "string", "enum": ["win32", "appx", "winget", "choco"] },
            "install_date": { "type": ["string", "null"], "format": "date-time" },
            "scope": { "type": "string", "enum": ["user", "machine"] }
          }
        }
      },
      "total": { "type": "integer" },
      "provider": { "type": "string" },
      "provider_version": { "type": ["string", "null"] }
    }
  },
  "error_schema": {
    "type": "object",
    "properties": {
      "error_code": { "type": "string", "enum": ["INVALID_REQUEST", "PROVIDER_UNAVAILABLE", "OBSERVATION_FAILED", "INSUFFICIENT_PRIVILEGE", "PARTIAL_OBSERVATION", "EVIDENCE_GENERATION_FAILED"] },
      "message": { "type": "string" },
      "operation_id": { "type": "string", "const": "OP-002" },
      "timestamp": { "type": "string", "format": "date-time" }
    }
  }
}
```

---

## Tool: tweak_state

```json
{
  "name": "tweak_state",
  "description": "Returns the current state of Windows tweak categories: privacy, telemetry, UI, services, scheduled tasks. Each tweak shows current value, expected value, and whether it is in the desired state. Provider must enforce read-only parameter binding — no tweak is applied or modified.",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "input_schema": {
    "type": "object",
    "properties": {
      "category": {
        "type": "string",
        "enum": ["privacy", "telemetry", "ui", "services", "tasks", "all"],
        "default": "all"
      },
      "tweak_ids": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Optional list of specific tweak IDs to query. If null, returns all tweaks in the category."
      }
    },
    "additionalProperties": false
  },
  "output_schema": {
    "type": "object",
    "required": ["operation_id", "timestamp", "tweaks", "provider"],
    "properties": {
      "operation_id": { "type": "string", "const": "OP-003" },
      "timestamp": { "type": "string", "format": "date-time" },
      "tweaks": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["id", "name", "category", "current_value", "in_desired_state"],
          "properties": {
            "id": { "type": "string" },
            "name": { "type": "string" },
            "category": { "type": "string", "enum": ["privacy", "telemetry", "ui", "services", "tasks"] },
            "current_value": {},
            "expected_value": {},
            "in_desired_state": { "type": "boolean" }
          }
        }
      },
      "provider": { "type": "string" },
      "provider_version": { "type": ["string", "null"] }
    }
  },
  "error_schema": {
    "type": "object",
    "properties": {
      "error_code": { "type": "string", "enum": ["INVALID_REQUEST", "PROVIDER_UNAVAILABLE", "OBSERVATION_FAILED", "UNSAFE_PROVIDER_PATH", "PARTIAL_OBSERVATION", "EVIDENCE_GENERATION_FAILED"] },
      "message": { "type": "string" },
      "operation_id": { "type": "string", "const": "OP-003" },
      "timestamp": { "type": "string", "format": "date-time" }
    }
  }
}
```

---

## Tool: provider_capabilities

```json
{
  "name": "provider_capabilities",
  "description": "Returns the capabilities and version of the active provider. Use this to discover what operations are available before making requests. Read-only, no system interaction beyond version queries.",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "input_schema": {
    "type": "object",
    "properties": {},
    "additionalProperties": false
  },
  "output_schema": {
    "type": "object",
    "required": ["operation_id", "timestamp", "provider_id", "supported_operations", "supported_domains"],
    "properties": {
      "operation_id": { "type": "string", "const": "OP-004" },
      "timestamp": { "type": "string", "format": "date-time" },
      "provider_id": { "type": "string" },
      "provider_version": { "type": ["string", "null"] },
      "supported_operations": {
        "type": "array",
        "items": { "type": "string", "enum": ["OP-001", "OP-002", "OP-003", "OP-005", "OP-006"] }
      },
      "supported_domains": {
        "type": "array",
        "items": { "type": "string" }
      },
      "max_package_query": { "type": ["integer", "null"] },
      "provider_specific": { "type": "object" }
    }
  },
  "error_schema": {
    "type": "object",
    "properties": {
      "error_code": { "type": "string", "enum": ["INVALID_REQUEST", "PROVIDER_UNAVAILABLE", "OBSERVATION_FAILED", "EVIDENCE_GENERATION_FAILED"] },
      "message": { "type": "string" },
      "operation_id": { "type": "string", "const": "OP-004" },
      "timestamp": { "type": "string", "format": "date-time" }
    }
  }
}
```

---

## Tool: update_status

```json
{
  "name": "update_status",
  "description": "Returns the current state of Windows Update: pending updates, installed hotfix history, and update service status. Does NOT trigger update client activation (no UsoClient/wuauclt invocation). Uses only WMI/log queries. Read-only, no install/scan/check-for-updates triggered.",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "input_schema": {
    "type": "object",
    "properties": {
      "include_history": {
        "type": "boolean",
        "default": false,
        "description": "Include installed update history (Get-HotFix). Default false."
      }
    },
    "additionalProperties": false
  },
  "output_schema": {
    "type": "object",
    "required": ["operation_id", "timestamp", "service_status", "pending_updates", "provider"],
    "properties": {
      "operation_id": { "type": "string", "const": "OP-005" },
      "timestamp": { "type": "string", "format": "date-time" },
      "service_status": { "type": "string" },
      "last_check": { "type": ["string", "null"], "format": "date-time" },
      "pending_updates": { "type": "integer" },
      "update_history": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["kb", "title"],
          "properties": {
            "kb": { "type": "string" },
            "title": { "type": "string" },
            "installed_on": { "type": ["string", "null"], "format": "date-time" },
            "result": { "type": "string" }
          }
        }
      },
      "provider": { "type": "string" },
      "provider_version": { "type": ["string", "null"] }
    }
  },
  "error_schema": {
    "type": "object",
    "properties": {
      "error_code": { "type": "string", "enum": ["INVALID_REQUEST", "PROVIDER_UNAVAILABLE", "OBSERVATION_FAILED", "UNSAFE_PROVIDER_PATH", "PARTIAL_OBSERVATION", "EVIDENCE_GENERATION_FAILED"] },
      "message": { "type": "string" },
      "operation_id": { "type": "string", "const": "OP-005" },
      "timestamp": { "type": "string", "format": "date-time" }
    }
  }
}
```

---

## Tool: system_configuration_summary

```json
{
  "name": "system_configuration_summary",
  "description": "Returns a consolidated read-only snapshot of system configuration: network adapters and IP config, firewall profile status, DNS servers, disk space, running services summary (user-scoped), and scheduled task overview. All queries are WMI/CIM reads — no service start/stop, no firewall modification.",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "input_schema": {
    "type": "object",
    "properties": {
      "sections": {
        "type": "array",
        "items": { "type": "string", "enum": ["network", "firewall", "storage", "services", "tasks"] },
        "description": "Select specific sections to query. If null, returns all."
      }
    },
    "additionalProperties": false
  },
  "output_schema": {
    "type": "object",
    "required": ["operation_id", "timestamp", "provider"],
    "properties": {
      "operation_id": { "type": "string", "const": "OP-006" },
      "timestamp": { "type": "string", "format": "date-time" },
      "network": {
        "type": "object",
        "properties": {
          "adapters": {
            "type": "array",
            "items": {
              "type": "object",
              "required": ["name", "status"],
              "properties": {
                "name": { "type": "string" },
                "status": { "type": "string", "enum": ["up", "down", "unknown"] },
                "ip_addresses": { "type": "array", "items": { "type": "string" } },
                "dns_servers": { "type": "array", "items": { "type": "string" } }
              }
            }
          }
        }
      },
      "firewall": {
        "type": "object",
        "properties": {
          "profiles": {
            "type": "object",
            "properties": {
              "domain": { "type": "boolean" },
              "private": { "type": "boolean" },
              "public": { "type": "boolean" }
            }
          }
        }
      },
      "storage": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["drive_letter", "size_gb", "free_gb"],
          "properties": {
            "drive_letter": { "type": "string" },
            "size_gb": { "type": "number" },
            "free_gb": { "type": "number" }
          }
        }
      },
      "services_count": { "type": "integer" },
      "tasks_count": { "type": "integer" },
      "provider": { "type": "string" },
      "provider_version": { "type": ["string", "null"] }
    }
  },
  "error_schema": {
    "type": "object",
    "properties": {
      "error_code": { "type": "string", "enum": ["INVALID_REQUEST", "PROVIDER_UNAVAILABLE", "OBSERVATION_FAILED", "INSUFFICIENT_PRIVILEGE", "PARTIAL_OBSERVATION", "EVIDENCE_GENERATION_FAILED"] },
      "message": { "type": "string" },
      "operation_id": { "type": "string", "const": "OP-006" },
      "timestamp": { "type": "string", "format": "date-time" }
    }
  }
}
```

---

## Security Invariants — Enforced at MCP Layer

```
WRITE_TOOLS_EXPOSED = 0
PLAN_TOOLS_EXPOSED = 0
ARBITRARY_POWERSHELL = 0
RAW_COMMAND_EXECUTION = 0
RAW_REGISTRY_WRITE = 0
RAW_SERVICE_CONTROL = 0
DOWNLOAD_AND_EXECUTE = 0
DYNAMIC_SCRIPT_EXECUTION = 0
PROVIDER_WRITE_PATH_REACHABLE = NO
ELEVATION_REQUIRED_BY_MCP = NO
OPERATION_INTENT = READ
RUNTIME_ENFORCEMENT = READ_ONLY
SIDE_EFFECT_EXPECTATION = NONE
SYSTEM_STATE_CHANGE = IMPOSSIBLE_BY_CONTRACT
```

> **Design invariant, not runtime proof.** These claims are contract-level for Gate 2C.
> Runtime assurance requires Gate 2D/2E implementation and negative testing.

---

## Normalized Error Codes

| Error Code | Meaning |
|---|---|
| `INVALID_REQUEST` | Malformed input, missing required fields, out-of-range values |
| `UNSUPPORTED_CAPABILITY` | Requested operation not supported by current provider |
| `PROVIDER_UNAVAILABLE` | Active provider cannot be reached or is not initialized |
| `OBSERVATION_FAILED` | Provider encountered an error while reading system state |
| `INSUFFICIENT_PRIVILEGE` | Requested observation requires elevation not available in current context |
| `UNSAFE_PROVIDER_PATH` | Provider attempted to execute a non-read-only path — operation blocked |
| `PARTIAL_OBSERVATION` | Provider returned partial results; some subsystems unavailable |
| `EVIDENCE_GENERATION_FAILED` | Provider could not generate required evidence metadata |
| `INTERNAL_ERROR` | Unexpected internal error in WCC-MCP core |

---

## Evidence Metadata (per tool invocation)

Every tool invocation produces:

```json
{
  "request_id": "<uuid>",
  "operation_id": "OP-00N",
  "timestamp": "<ISO8601>",
  "operation_intent": "READ",
  "runtime_enforcement": "READ_ONLY",
  "privilege_context": "USER_CONTEXT",
  "provider": "<provider_id>",
  "provider_version": "<string | null>",
  "result": "success | failure | partial",
  "failure_code": "<error_code | null>",
  "side_effect_observed": "<boolean>",
  "evidence_hash": "<sha256 of normalized output>"
}
```

> **Non-repudiation caveat:** SHA-256 hash provides integrity, not non-repudiation.
> Do not claim cryptographic signing or legal evidence weight from a simple hash.
