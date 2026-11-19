# ZTD Architecture Notes

**Specification baseline:** Zero-Trust Defender v13.1 (Draft for Review)  
**Document status:** Proposed architecture; implementation not yet verified.

## 1. Architectural goals

The proposed design separates collection, deterministic detection, AI-assisted investigation, response orchestration, persistence and management. This separation is intended to make components independently testable and allow reduced-capability operation on offline or resource-constrained hosts.

## 2. Logical components

### Endpoint agent

Responsibilities proposed in the PRD:

- Collect configured file, process, CPU, memory and network telemetry.
- Inspect new or modified files using hashes, PE metadata and available YARA rules.
- Monitor platform-specific host events through ETW/WMI or eBPF where available.
- Buffer events locally when the central server cannot be reached.
- Communicate with the server over mutually authenticated TLS (mTLS).
- Report agent health, platform, architecture and runtime profile.

Agents should run with the minimum privileges required by the enabled sensors. Privileged collection and response operations should be isolated from ordinary parsing and UI code.

### Deterministic detection layer

The first-stage detection layer is intended to use:

- Hash and reputation lookups.
- PE metadata and entropy heuristics.
- YARA signatures.
- Behavioural rules and anomaly scoring.
- DNS and network-flow heuristics.
- Optional trained models when validated artifacts are available.

Deterministic detections should remain available when no LLM is configured or network connectivity is absent.

### Multi-LLM orchestration layer

The v13.1 design proposes adapters for OpenAI, Anthropic and xAI APIs, alongside local runtimes such as Ollama, llama.cpp/GGUF and Transformers.

The orchestration layer may provide provider selection, failover, cost-aware routing, structured tool calls, model comparison and local-only inference in air-gapped mode. Provider credentials must be held outside the repository and never embedded in prompts or logs.

LLM output must be treated as untrusted input. Tool calls should pass schema validation, authorization checks, policy evaluation and audit logging before any operation is attempted.

### Response and playbook engine

The proposed response layer includes quarantine, firewall blocking, process termination and scoped persistence cleanup. It should enforce:

1. An allowlisted action catalogue.
2. Explicit target and scope validation.
3. Least-privilege execution.
4. Configurable confidence and consensus requirements.
5. Human approval for configured high-impact actions.
6. Dry-run support and pre-action state capture where possible.
7. Action result verification, audit events and rollback procedures where feasible.

A model must not be able to bypass these controls by generating a tool call or playbook.

### Management plane

The server/dashboard is intended to provide authentication, role-based access, agent inventory, alert and incident views, audit search, reporting, feedback and SOC co-pilot access. The PRD proposes Flask or FastAPI and SQLite as an initial persistence option.

The dashboard should bind to localhost by default during development. Remote deployment requires hardened authentication, TLS, secure session handling and network access controls.

### Persistence and integrations

The PRD proposes SQLite plus structured JSON logs, offline event buffering, configurable retention and schema migrations. Optional integrations include MISP, VirusTotal, AbuseIPDB, Splunk HEC, ELK, TheHive and SMTP.

External integrations should use bounded retries, rate-limit-aware backoff, input validation and clear failure states. Offline caches need timestamps and provenance so stale intelligence is not mistaken for current data.

## 3. Event flow

1. An agent observes a configured host or network event.
2. The agent normalises the event and attaches source, timestamp and host context.
3. Local deterministic checks run where supported.
4. The event is stored locally and forwarded to the server when connected.
5. The server enriches and correlates events into alerts or incidents.
6. Optional AI analysis receives only the context allowed by data-handling policy.
7. A recommendation is returned as structured data and validated.
8. The response policy decides whether to reject, request approval, dry-run or execute an allowed action.
9. The result, actor, evidence and rollback status are recorded.

## 4. Trust boundaries

| Boundary | Required controls |
|---|---|
| Agent to server | mTLS, agent identity, replay protection, schema validation |
| Server to model provider | Explicit provider configuration, secret isolation, data minimisation |
| Model to tools | Strict schemas, allowlisted tools, authorization and policy checks |
| Dashboard to API | Authentication, RBAC, CSRF/session protections and rate limits |
| Threat-intel source to detection | Provenance, TTL, validation and safe parsing |
| Update package to runtime | Signature/hash verification and controlled rollout |

## 5. Offline and degraded operation

Air-gapped operation is a design objective. A disconnected installation should continue using local telemetry, deterministic rules, local persistence and locally available models. Cloud API features and live reputation lookups must be disabled when offline. Approved update packages may be imported through a controlled, verified transfer process.

If optional components such as eBPF, ETW, Scapy, GPU acceleration or a local model runtime are unavailable, the agent should report the missing capability and continue with a clearly documented reduced feature set.

## 6. Runtime profiles

- **Full:** broadest supported collection and analysis on higher-resource systems.
- **Balanced:** moderate concurrency and model/resource use.
- **Lightweight:** streaming scans, bounded queues, reduced concurrency and optional quantized local models.

Profile selection should be observable and overridable by an administrator. Resource limits must be measured on supported hardware rather than assumed.

## 7. Key architecture decisions still requiring validation

- Agent/server protocol and event schema versioning.
- Exact privilege boundaries for each sensor and response action.
- Local database encryption and key management.
- Queue limits, backpressure and offline replay semantics.
- Model-provider adapter contracts and structured-output validation.
- Safe quarantine storage and recovery workflow.
- Cross-platform packaging and update signing.
- Measured performance, scale and compatibility results.

This document describes the intended design from the v13.1 PRD. It is not evidence that these components have been implemented or security-tested.
