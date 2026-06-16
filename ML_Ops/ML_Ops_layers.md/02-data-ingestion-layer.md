# Threat Model: Data Ingestion Layer

## Scope

This model covers the ingestion layer that reads raw data from the landing zone, validates it, captures lineage, applies policy checks, quarantines rejected data, and writes accepted data into controlled ingestion storage.

Out of scope: feature engineering, model training, model registry, model serving, inference, and downstream analytics consumption.

## Assets

| Asset | Security need |
| --- | --- |
| Raw landing data | Integrity, controlled access |
| Ingestion jobs and orchestration | Integrity, availability, least privilege |
| Parser and validation code | Integrity, supply chain security |
| Schemas and data contracts | Integrity, version control |
| Quarantine storage | Isolation, auditability, confidentiality |
| Accepted ingestion storage | Integrity, lineage, access control |
| Ingestion logs and metrics | Integrity, non-repudiation |
| Secrets and service identities | Confidentiality, least privilege |

## Actors

| Actor | Description |
| --- | --- |
| Ingestion service | Automated job, stream processor, or workflow reading from landing. |
| Data engineer | Maintains ingestion logic, schemas, and operational rules. |
| Security or compliance reviewer | Reviews policy violations, quarantine events, and audit evidence. |
| Compromised landing source | Sends malicious, malformed, or poisoned data that reaches ingestion. |
| Malicious insider | Attempts to bypass validation, alter schemas, or approve unsafe data. |
| Attacker controlling dependency | Compromises parser library, container image, or ingestion dependency. |

## Trust Boundaries

| Boundary | Description | Primary risk |
| --- | --- | --- |
| Landing zone to ingestion runtime | Untrusted raw data enters processing code. | Parser exploit, malformed input, resource exhaustion |
| Ingestion runtime to secrets and metadata stores | Runtime accesses credentials and lineage systems. | Credential theft, unauthorized metadata changes |
| Validation/quarantine to accepted storage | Data changes trust level. | Policy bypass, poisoned data acceptance |
| Developer control plane to ingestion deployment | Code and configuration changes are promoted. | Supply chain compromise, unauthorized schema changes |

## Data Flow

1. Ingestion service discovers or receives notification of a new landing object, message, or batch.
2. Service verifies source metadata, freshness, checksum or digest, size, type, and schema version.
3. Data is parsed in a constrained runtime.
4. Validation checks run against schema, quality thresholds, policy rules, malware scanning results, and source contract.
5. Rejected data is written to quarantine with reason codes and restricted access.
6. Accepted data is written to controlled ingestion storage with lineage metadata.
7. Audit logs and metrics are emitted for every accept, reject, retry, and failure.

## Threats and Controls

| ID | STRIDE | Threat | Impact | Recommended controls |
| --- | --- | --- | --- | --- |
| ING-01 | Tampering | Ingestion accepts data with missing, forged, or mismatched metadata. | Incorrect lineage and potential poisoned data acceptance. | Metadata validation, source-to-schema binding, checksum verification, reject missing mandatory fields. |
| ING-02 | Tampering | Malformed files exploit parser behavior or bypass validation. | Runtime compromise, data corruption, or unsafe records accepted. | Sandboxed parsing, dependency patching, strict parsers, fuzz testing, file type verification. |
| ING-03 | Denial of service | Large, deeply nested, compressed, or high-cardinality inputs exhaust resources. | Ingestion outage, delayed feeds, cost spike. | Size limits, decompression ratio limits, timeouts, memory limits, streaming parsers, per-source quotas. |
| ING-04 | Elevation of privilege | Ingestion runtime has broad access to landing, accepted storage, secrets, or control plane. | Compromise spreads beyond ingestion. | Least-privilege service roles, separate read/write identities, secret scoping, network egress controls. |
| ING-05 | Tampering | Schema or validation rules are changed to allow unsafe data. | Data quality and security controls are bypassed. | Version-controlled schemas, code review, change approval, signed releases, policy-as-code tests. |
| ING-06 | Repudiation | Ingestion outcomes are not logged or logs can be modified. | Cannot investigate accepted, rejected, or altered data. | Append-only logs, correlation IDs, immutable audit storage, job run IDs, log integrity controls. |
| ING-07 | Information disclosure | Rejected data in quarantine exposes sensitive or regulated data to broad audiences. | Privacy breach and compliance failure. | Quarantine access restrictions, encryption, data classification, redacted previews, review workflow controls. |
| ING-08 | Tampering | Operator manually moves quarantined data into accepted storage without approval. | Known bad data enters downstream systems. | Segregation of duties, approval workflow, break-glass logging, deny direct writes to accepted zones. |
| ING-09 | Spoofing | Fake ingestion job or unauthorized workflow reads landing data. | Data theft or unauthorized processing. | Workload identity, job allowlists, signed deployments, runtime attestation where available. |
| ING-10 | Information disclosure | Ingestion logs include raw sensitive values, credentials, or full payloads. | Sensitive data leaks through observability systems. | Structured logging, redaction, secret detection, log sampling policies, restricted log access. |
| ING-11 | Denial of service | Poisoned batch causes repeated retries and blocks later valid data. | Pipeline stalls and freshness objectives fail. | Dead-letter queues, bounded retries, batch isolation, partial failure handling, alerting. |
| ING-12 | Tampering | Dependency, container image, or package used by ingestion is compromised. | Attacker controls ingestion behavior. | Image signing, SBOMs, dependency scanning, pinned versions, provenance checks, restricted registries. |
| ING-13 | Tampering | Duplicate or replayed batches are processed multiple times. | Biased data, incorrect counts, or downstream skew. | Idempotency keys, batch sequence tracking, deduplication, exactly-once or effectively-once design. |
| ING-14 | Elevation of privilege | Validation code executes source-provided scripts, formulas, or macros. | Remote code execution in ingestion runtime. | Disable active content, parse as data only, block macros/scripts, use safe libraries and sandboxing. |

## Required Security Requirements

- Ingestion must treat all landing data as untrusted until validation completes.
- Accepted data must be separated from raw and quarantined data by storage location and access policy.
- Every accepted or rejected batch must have lineage metadata and a validation outcome.
- Parser runtimes must enforce resource limits and must not execute active content from source data.
- Schema and validation rule changes must follow reviewed, version-controlled release processes.
- Ingestion service identities must use least privilege and separate read/write permissions.
- Quarantine release must require explicit approval and produce an auditable record.

## Detection and Monitoring

- Sudden increase in validation failures or quarantine volume.
- Parser crashes, memory pressure, long runtimes, or retry storms.
- Schema mismatch, unknown schema version, or unexpected file type.
- Accepted data without complete lineage or validation record.
- Manual access to quarantine or accepted ingestion storage.
- Direct writes to accepted storage outside the ingestion service identity.
- Dependency vulnerability, unsigned image, or unapproved deployment.
- Logs containing secrets, raw PII, or unusually large payload fragments.

## Validation Checklist

- [ ] Are landing, quarantine, and accepted ingestion stores separated by access policy?
- [ ] Does ingestion reject missing source identity, checksum, schema version, or arrival metadata?
- [ ] Are parsers constrained by CPU, memory, file size, timeout, and decompression limits?
- [ ] Are schema and rule changes reviewed and versioned?
- [ ] Are all accepted and rejected records traceable to source and job run?
- [ ] Can a poisoned batch fail without blocking unrelated valid batches?
- [ ] Is quarantine release controlled by approval and audit logging?
- [ ] Are ingestion logs redacted and protected from unauthorized access?

