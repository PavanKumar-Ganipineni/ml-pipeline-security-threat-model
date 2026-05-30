# Additional ML Pipeline Security Additions

The original draft covered core ML security areas: supply chain scanning, model binary scanning, safe model serialization, RONI filtering, activation clustering, robust optimization, adversarial training, DP-SGD, output abstraction, and operational success metrics.

The following ML-specific items were added to close common threat-model gaps.

## Added Threats

- **Dataset provenance failure:** Training data enters from an untrusted or unapproved source.
- **Dataset version confusion:** Training jobs accidentally use stale, poisoned, or unauthorized snapshots.
- **Unauthorized label mutation:** Labels are changed after approval, shifting model behavior.
- **Golden Baseline Dataset tampering:** The reference dataset used for RONI and regression testing is modified.
- **Feature store poisoning:** Derived feature values are altered before training or inference.
- **Training data leakage:** Sensitive values leak through notebooks, experiment trackers, feature tables, logs, or exported datasets.
- **Model registry tampering:** A validated model is replaced, downgraded, or modified after approval.
- **Unauthorized model promotion:** A model is deployed without passing robustness, privacy, and artifact checks.
- **Rollback to vulnerable model:** An old rejected model is redeployed during rollback.
- **CI/CD credential theft:** Cloud keys, registry tokens, or service account credentials leak from ML automation.
- **Malicious pipeline step injection:** A pipeline definition is changed to bypass scanning or exfiltrate data.
- **Experiment tracker leakage:** Metadata tools expose sensitive samples, paths, parameters, or prediction outputs.
- **Silent data drift:** Production data shifts without detection.
- **Feedback loop poisoning:** Production feedback is manipulated and later used for retraining.
- **Monitoring blind spots:** Telemetry does not detect extraction, probing, evasion, or poisoning attempts.

## Added Mitigations

- Dataset source, owner, license, approval status, and checksum tracking.
- Immutable versioning for training, validation, test, and golden baseline datasets.
- Label mutation auditing with reviewer, timestamp, previous value, new value, and reason.
- Least-privilege controls for feature store reads, writes, approvals, and deployments.
- PII and secret scanning for datasets, features, notebook outputs, and logs.
- Feature freshness, integrity, row count, null rate, and distribution checks.
- Signed model artifacts and registry checksums.
- Immutable approved model versions.
- Production promotion gates for scanning, robustness testing, privacy checks, and approvals.
- Rollback restrictions to currently approved model versions.
- Secret scanning across repositories, notebooks, pipeline YAML files, and logs.
- Short-lived OIDC-based CI/CD authentication.
- Pipeline definition review for training, evaluation, deployment, and retraining steps.
- Experiment tracking hygiene to prevent sensitive logging.
- Drift monitoring for features, predictions, confidence values, and class balance.
- Quarantine zones for production feedback before retraining.
- Abuse detection for high-volume queries, confidence probing, query sweeps, and near-duplicate inputs.
- Security regression testing before every retrained model promotion.

