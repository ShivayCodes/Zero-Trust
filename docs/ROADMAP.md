# ZTD Roadmap and Validation Plan

**Baseline:** Zero-Trust Defender v13.1 PRD  
**Status:** Proposed milestones; no milestone is marked complete because this repository currently contains documentation only.

## Milestone 0 — Repository and engineering foundation

- [ ] Define supported Python versions and dependency policy.
- [ ] Establish package layout, configuration format and typed interfaces.
- [ ] Add linting, formatting, static checks and a CI test workflow.
- [ ] Define secure logging rules and secret-management policy.
- [ ] Write initial threat model and security disclosure process.

**Exit evidence:** reproducible development setup, passing CI, documented architecture decisions and no secrets in repository history.

## Milestone 1 — Local agent and event pipeline

- [ ] Implement a minimal agent lifecycle and configuration loader.
- [ ] Define a versioned event schema with host and sensor provenance.
- [ ] Add bounded local event buffering and clean shutdown.
- [ ] Add platform capability detection and health reporting.
- [ ] Test operation without internet connectivity.

**Exit evidence:** unit/integration tests, sample event records, resource measurements and offline test results.

## Milestone 2 — Deterministic detection

- [ ] Add file hashing and safe metadata inspection.
- [ ] Integrate YARA with controlled rule loading.
- [ ] Add selected process, filesystem and DNS behavioural rules.
- [ ] Add alert severity, evidence and deduplication fields.
- [ ] Build benign test fixtures and isolated detection tests.

**Exit evidence:** reproducible detection tests, false-positive analysis, performance results and documented sensor limitations.

## Milestone 3 — Server, storage and dashboard

- [ ] Implement authenticated agent enrollment and mTLS communication.
- [ ] Add server-side event ingestion and deduplicated replay.
- [ ] Add persistence, migrations and retention controls.
- [ ] Add dashboard/API with role-based access.
- [ ] Add audit log search and export.

**Exit evidence:** authentication tests, authorization tests, migration tests, API tests and deployment documentation.

## Milestone 4 — Governed response

- [ ] Define an allowlisted response-action interface.
- [ ] Implement dry-run mode and scoped quarantine in a lab.
- [ ] Add firewall action adapters with post-action verification.
- [ ] Add approval policies for high-impact operations.
- [ ] Add action audit records and rollback/recovery procedures.

**Exit evidence:** negative authorization tests, rollback drills, failure-injection tests and documented operator procedures. No autonomous production response should be enabled before these gates pass.

## Milestone 5 — Threat intelligence and external integrations

- [ ] Add optional MISP and reputation-provider adapters.
- [ ] Add cache TTL, provenance and rate-limit-aware backoff.
- [ ] Add SIEM and case-management export adapters.
- [ ] Verify that integrations fail safely when unavailable.
- [ ] Validate privacy settings and outbound data minimisation.

**Exit evidence:** integration tests with mocks or authorised test tenants, offline cache tests and data-flow documentation.

## Milestone 6 — Local and cloud LLM assistance

- [ ] Define a provider-neutral model adapter interface.
- [ ] Add local inference integration and offline model verification.
- [ ] Add optional cloud provider adapters with secrets outside source control.
- [ ] Add structured response schemas and tool-call validation.
- [ ] Add cost/latency budgets, provider failover and model-consensus tests.
- [ ] Keep deterministic detection functional without an LLM.

**Exit evidence:** prompt-injection tests, tool authorization tests, data-exposure review, model failure tests and local-only network verification.

## Milestone 7 — Cross-platform and scale validation

- [ ] Validate Windows, Linux and macOS agent behaviour.
- [ ] Validate x86_64 and ARM64 packaging.
- [ ] Test virtualization-aware and Lightweight profiles.
- [ ] Test Raspberry Pi-class hardware.
- [ ] Test air-gapped installation, updates and recovery.
- [ ] Measure CPU, memory, scan rate, MTTD, MTTR and server capacity.

**Exit evidence:** published compatibility matrix with hardware/software versions, reproducible benchmarks and known limitations.

## PRD targets are not measured results

The v13.1 PRD proposes the following targets:

| Metric | Proposed target |
|---|---:|
| Mean time to detect | Under 10 seconds |
| Mean time to respond | Under 18 seconds |
| Post-consensus AI false-positive rate | Under 0.9% |
| Endpoints per server | At least 8,000 |
| Incidents resolved without human intervention | At least 82% |
| Full-profile agent CPU overhead | At most 3.5% |
| Lightweight/Pi agent CPU overhead | At most 12% |
| Core capability retention in air-gapped mode | 100% |

These figures must not be presented as achieved until backed by repeatable benchmark methodology, test data and published results.

## Release gate

A release should not be described as production-ready until its supported-platform tests, security review, operational recovery tests, performance benchmarks, documentation and known limitations are published. High-impact response actions must remain policy-controlled and disabled by default until their safety and recovery behaviour have been validated.
