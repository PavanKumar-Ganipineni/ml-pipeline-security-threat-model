# Threat Model: Data Sources to Landing Zone

## Scope

This model covers data movement from different source systems into the landing zone. It includes transfer channels, source authentication, data submission, landing storage, and initial metadata capture.

Out of scope: ingestion parsing, feature engineering, training, model serving, and downstream analytics consumption.

## Assets

| Asset | Security need |
| --- | --- |
| Source data files, payloads, events, and extracts | Integrity, confidentiality, availability |
| Source identity and credentials | Confidentiality, non-repudiation |
| Landing zone storage | Integrity, access control, durability |
| Transfer channels | Confidentiality, integrity, endpoint authenticity |
| Arrival metadata | Integrity, auditability, lineage |
| Source allowlists and contracts | Integrity, availability |

## Actors

| Actor | Description |
| --- | --- |
| Internal source system | Approved enterprise system sending data to the landing zone. |
| External partner or vendor | Third party providing approved data feeds. |
| Pipeline operator | Engineer or service owner managing source onboarding and landing configuration. |
| Malicious external actor | Unapproved actor attempting to inject, alter, or intercept data. |
| Compromised source | Legitimate source system or credential used by an attacker. |
| Insider | User with some legitimate access attempting unauthorized submission or modification. |

## Trust Boundaries

| Boundary | Description | Primary risk |
| --- | --- | --- |
| Source environment to network transfer | Data leaves the source control plane. | Interception, tampering, spoofing |
| Transfer endpoint to landing zone | Data is accepted by the pipeline boundary. | Unauthorized submission, replay, misrouting |
| Landing zone to pipeline services | Raw data becomes available to internal services. | Premature trust, lateral movement, data leakage |
| Operator access to landing controls | Humans configure source access and routing. | Misconfiguration, excessive privilege |

## Data Flow

1. Source system prepares data according to an approved data contract.
2. Source authenticates to the transfer endpoint or writes to an approved landing path.
3. Data is transmitted over an approved channel.
4. Landing zone stores the raw object, message, or payload.
5. Metadata is recorded, including source identity, arrival time, object path, checksum or digest, size, schema version, and transfer status.
6. The landing object is made available only to ingestion services and restricted operators.

## Threats and Controls

| ID | STRIDE | Threat | Impact | Recommended controls |
| --- | --- | --- | --- | --- |
| LZ-01 | Spoofing | Unauthorized source submits data by impersonating an approved source. | Poisoned or fraudulent data enters the pipeline. | Strong source authentication, mTLS or signed requests, short-lived credentials, source allowlists, per-source landing paths. |
| LZ-02 | Tampering | Data is modified in transit before reaching the landing zone. | Downstream training data may be corrupted or poisoned. | TLS, object signing, checksums, message authentication codes, immutable arrival metadata. |
| LZ-03 | Tampering | Attacker overwrites or deletes objects in the landing zone. | Loss of data integrity and availability. | Object versioning, write-once policies, least-privilege write roles, deny delete by default, immutable retention where required. |
| LZ-04 | Repudiation | Source denies sending a file, event, or payload. | Weak auditability and incident investigation. | Source-specific credentials, signed manifests, append-only audit logs, request IDs, synchronized timestamps. |
| LZ-05 | Information disclosure | Sensitive data is exposed during transfer or while stored in landing. | Privacy breach, regulatory exposure, credential leakage. | Encryption in transit and at rest, private endpoints, restricted bucket/container policies, secrets scanning, data classification tags. |
| LZ-06 | Denial of service | Source or attacker floods landing with excessive volume or oversized files. | Storage exhaustion, ingestion delays, cost spike. | Per-source quotas, rate limits, file size limits, backpressure, alerting on arrival anomalies. |
| LZ-07 | Elevation of privilege | Landing write credentials allow read, delete, or admin actions. | Attacker gains access beyond data submission. | Separate write-only roles, scoped IAM policies, credential rotation, policy-as-code checks. |
| LZ-08 | Tampering | Replay of old but valid data is accepted as new. | Model inputs become stale or intentionally biased. | Nonces, timestamps, manifest sequence numbers, duplicate detection, freshness checks. |
| LZ-09 | Information disclosure | Data is routed to the wrong landing path or tenant partition. | Cross-tenant or cross-domain leakage. | Source-to-path binding, tenant-aware access controls, automated routing tests, deny wildcard writes. |
| LZ-10 | Tampering | Malicious compressed archive, nested file, or malformed object is landed. | Parser exploit or resource exhaustion during ingestion. | Treat landing as untrusted, archive restrictions, malware scanning, decompression limits, quarantine before parsing. |
| LZ-11 | Repudiation | Missing or mutable metadata prevents source lineage reconstruction. | Incident response cannot identify affected data. | Mandatory metadata schema, append-only metadata store, checksum capture, immutable event logs. |
| LZ-12 | Denial of service | Dependency outage in partner transfer, DNS, identity, or private link prevents delivery. | Fresh data is unavailable for ingestion. | Retry policies, source SLAs, health checks, alternate transfer path for critical feeds, freshness alerts. |

## Required Security Requirements

- Every source must have an owner, approved data contract, source identifier, authentication method, and allowed landing destination.
- Landing write permissions must not include broad read, delete, or administrative access.
- Raw landing data must be encrypted in transit and at rest.
- Arrival metadata must be captured before ingestion processing begins.
- Landing objects must be immutable or versioned enough to support investigation.
- Data from unapproved sources must be rejected or quarantined.
- Access to raw landing data must be limited to ingestion services and authorized operators.

## Detection and Monitoring

- Unexpected source identifier, IP, certificate, account, or service principal.
- Arrival volume, size, frequency, or file type outside source baseline.
- Repeated failed authentication or authorization attempts.
- Data landing outside approved path patterns.
- Missing checksum, manifest, schema version, or source metadata.
- Delete, overwrite, permission change, or public exposure events on landing storage.
- Stale feeds, duplicate batches, or replayed sequence numbers.

## Validation Checklist

- [ ] Is each source mapped to a named owner and approved data contract?
- [ ] Are source credentials scoped to only the required landing action?
- [ ] Are landing paths source-specific and protected from cross-source writes?
- [ ] Are object versioning, retention, or immutability controls enabled where needed?
- [ ] Is raw data blocked from downstream direct consumption?
- [ ] Are transfer failures, unexpected volume, and missing metadata alerted?
- [ ] Can security teams reconstruct who sent what, when, and from where?

