# ML Pipeline Risk Register

| ID | Pipeline Phase | Threat | Impact | Likelihood | Risk | Primary Mitigation |
| :--- | :--- | :--- | :---: | :---: | :---: | :--- |
| ML-001 | Supply Chain | Dependency hijacking through typosquatting or dependency confusion | High | Medium | High | SCA scanning, dependency pinning, approved package registry |
| ML-002 | Supply Chain | Critical or High CVEs in ML dependencies | High | Medium | High | Snyk CI/CD gate and build blocking |
| ML-003 | Supply Chain | Vulnerable GPU/CUDA base image | High | Medium | High | Prisma Cloud or Snyk Container scanning, GCP Artifact Registry Container Analysis |
| ML-004 | Supply Chain | Malicious `.pkl` or `.joblib` model artifact executes system commands | Critical | Medium | Critical | ModelScan, Picklescan, safer formats such as `safetensors` and ONNX |
| ML-005 | Training Data | Outlier-driven denial of service degrades global model performance | Medium | High | High | Data validation, RONI filtering, robust loss functions |
| ML-006 | Training Data | Backdoor or trojan trigger causes targeted misclassification | Critical | Medium | Critical | Activation clustering, quarantined review, adversarial validation |
| ML-007 | Training Data | Clean-label poisoning shifts decision boundaries | High | Medium | High | RONI filtering, label audits, trimmed optimization |
| ML-008 | Training Data | Golden Baseline Dataset tampering weakens security gates | Critical | Low | High | Immutable dataset versioning, restricted write access, checksums |
| ML-009 | Feature Store | Feature value poisoning affects training or inference | High | Medium | High | Least-privilege writes, freshness checks, integrity checks |
| ML-010 | Feature Store | Sensitive data leakage through features, notebooks, or logs | High | Medium | High | PII scanning, log controls, experiment tracking hygiene |
| ML-011 | Model Registry | Approved model artifact is replaced or modified after validation | Critical | Low | High | Signed artifacts, checksums, immutable registry versions |
| ML-012 | Deployment | Unauthorized model promotion to production | Critical | Medium | Critical | Promotion gates, approval workflows, CI/CD policy checks |
| ML-013 | Deployment | Rollback to vulnerable or rejected model version | High | Low | Medium | Rollback allowlist restricted to currently approved versions |
| ML-014 | CI/CD | Pipeline credential theft enables artifact or data access | Critical | Medium | Critical | Secret scanning, OIDC-based workload identity, least privilege |
| ML-015 | CI/CD | Malicious pipeline step bypasses scanning or exfiltrates data | Critical | Low | High | Pipeline definition review, policy-as-code, isolated job identities |
| ML-016 | Inference | Evasion attack manipulates live predictions | High | Medium | High | Adversarial training, denoising layer, robustness testing |
| ML-017 | Inference | Membership inference exposes whether a user was in training data | High | Medium | High | DP-SGD, confidence masking, output abstraction |
| ML-018 | Inference | Model extraction through high-volume structured queries | High | Medium | High | Rate limiting, hard labels or coarse confidence intervals, abuse detection |
| ML-019 | Monitoring | Silent data drift weakens model reliability and security posture | Medium | High | High | Drift monitoring, retraining gates, security regression tests |
| ML-020 | Retraining | Feedback loop poisoning contaminates future training data | High | Medium | High | Feedback quarantine, RONI filtering, source reputation checks |

