# ML Pipeline Security Threat Model

This repository contains a security threat model and defensive blueprint for predictive machine learning pipelines.

The model focuses strictly on ML pipeline security across software supply chain intake, model artifact ingestion, training data protection, robust training, live inference defense, endpoint privacy, operational controls, and continuous monitoring.

## Contents

- [threat-model.md](threat-model.md) - Full ML pipeline threat model and security blueprint
- [controls/security-controls.md](controls/security-controls.md) - Consolidated ML pipeline security controls checklist
- [examples/risk-register.md](examples/risk-register.md) - Risk register with threats, impact, likelihood, and mitigations
- [ADDITIONS.md](ADDITIONS.md) - Additional ML-specific threats and mitigations added beyond the original draft

## Pipeline Phases Covered

1. Software and model supply chain defense
2. Inbound data and training phase defense
3. Live inference and endpoint privacy defense
4. Operational controls and success metrics
5. Additional ML pipeline security gaps

## Intended Use

Use this repository as a baseline for:

- MLOps security reviews
- ML pipeline architecture reviews
- CI/CD and artifact registry guardrails
- Training data protection
- Model registry and deployment approval workflows
- Inference endpoint hardening
- ML-specific risk assessment

## Scope

This threat model is limited to ML pipeline security. It does not attempt to cover general enterprise security outside the ML lifecycle except where it directly affects ML assets, pipelines, models, datasets, registries, or inference endpoints.

