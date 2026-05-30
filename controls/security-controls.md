# ML Pipeline Security Controls Checklist

## Phase 1: Software & Model Supply Chain Defense

- [ ] Integrate Snyk or equivalent SCA into CI/CD for `requirements.txt`, lockfiles, and ML package dependencies.
- [ ] Block builds containing Critical or High CVEs unless an approved exception exists.
- [ ] Scan notebook and pipeline base containers using Prisma Cloud Compute, Snyk Container, or equivalent tooling.
- [ ] Enable continuous background scanning using GCP Artifact Registry Container Analysis.
- [ ] Scan externally sourced model artifacts using ModelScan, Picklescan, or equivalent tooling.
- [ ] Block unsafe serialized artifacts that contain payload indicators such as `os.system` or `subprocess`.
- [ ] Enforce zero-code-execution model formats such as `safetensors` or ONNX where supported.
- [ ] Store only approved packages, containers, and model artifacts in the internal artifact registry.

## Phase 2: Inbound Data & Training Phase Defense

- [ ] Run RONI filtering before appending new data streams to primary training pools.
- [ ] Validate candidate datasets against a locked Golden Baseline Dataset.
- [ ] Quarantine data batches that cause global error spikes.
- [ ] Run activation clustering against incoming training datasets.
- [ ] Investigate dense, irregular sub-clusters inside individual classes as possible backdoor indicators.
- [ ] Use robust losses such as Huber Loss for applicable tasks.
- [ ] Drop or review the top 2% to 5% highest-loss training samples during each epoch where appropriate.
- [ ] Version training, validation, test, and golden baseline datasets immutably.

## Phase 3: Live Inference & Endpoint Privacy Defense

- [ ] Use adversarial training with ART or equivalent tooling.
- [ ] Generate FGSM or other adversarial examples against internal architectures during robustness testing.
- [ ] Train sensitive final model layers with DP-SGD where privacy risk requires it.
- [ ] Use gradient clipping and calibrated Gaussian noise injection through TensorFlow Privacy, Opacus, or equivalent tooling.
- [ ] Prevent deployed APIs from returning raw floating-point prediction arrays.
- [ ] Return hard class labels or coarse confidence intervals where feasible.
- [ ] Apply rate limits to inference endpoints.
- [ ] Monitor for model extraction query patterns.

## Dataset Lineage & Provenance

- [ ] Track dataset source, owner, collection date, license, approval status, and checksum.
- [ ] Make approved datasets immutable.
- [ ] Log all label changes with reviewer, timestamp, old value, new value, and reason.
- [ ] Restrict write access to golden baseline datasets.

## Feature Store & Access Control

- [ ] Separate read, write, approve, and deploy permissions.
- [ ] Scan datasets, features, notebooks, and logs for PII and secrets.
- [ ] Validate feature freshness, row counts, null rates, and distribution profiles.
- [ ] Run training jobs under scoped service accounts.

## Model Registry & Deployment Integrity

- [ ] Sign model artifacts before registry promotion.
- [ ] Store model checksums with each approved version.
- [ ] Require promotion gates before production deployment.
- [ ] Make approved model versions immutable.
- [ ] Allow rollback only to models that still satisfy current security policy.

## CI/CD, Secrets, and Pipeline Orchestration

- [ ] Scan repositories, notebooks, pipeline definitions, and logs for secrets.
- [ ] Use short-lived OIDC-based workload identity instead of long-lived static keys.
- [ ] Require review for pipeline definition changes.
- [ ] Prevent experiment tracking systems from logging raw samples, secrets, full prediction arrays, or sensitive feature values.

## Monitoring, Drift, and Retraining

- [ ] Monitor feature drift, prediction drift, confidence drift, and class balance changes.
- [ ] Quarantine production feedback before retraining.
- [ ] Detect high-volume querying, query sweeps, confidence probing, and repeated near-duplicate inputs.
- [ ] Run security regression tests before promoting retrained models.

