# Infrastructure Security Automation

![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Ansible](https://img.shields.io/badge/Ansible-Modern-red.svg)
![Docker](https://img.shields.io/badge/Docker-20.10+-blue.svg)
![GitLab CI](https://img.shields.io/badge/GitLab_CI-Pipeline-orange.svg)
![Trivy](https://img.shields.io/badge/Trivy-Security_Scanner-green.svg)
![IaC](https://img.shields.io/badge/IaC-Infrastructure_as_Code-purple.svg)

Production-grade **Platform Security / DevSecOps** reference implementation.

This repository demonstrates how to **reduce container risk in production** with:
- automated CVE scanning + safe rollouts (healthchecks + rollback),
- CI/CD security gates (build/test/deploy with blocking checks),
- secrets management (Ansible Vault),
- auditability (ticketed change trail).

> This is a **reference implementation** showcasing production-grade patterns (gates, rollback, traceability) on a demo environment.

## Overview

| Metric | Result |
|--------|--------|
| **Vulnerability Reduction** | 29 → 12 critical CVEs (58% reduction) |
| **Deployment Downtime** | < 20 seconds (pre-pull strategy) |
| **Pipeline Protection** | Syntax errors blocked before production |
| **Audit Trail** | Full traceability via automated ticketing |

> **Scope**: measured on the demo workload (nginx container) using Trivy "CRITICAL" severity before/after automated update & redeploy.

## Evaluate in 10 minutes

1. **CVE lifecycle**: run `scan_vulnerabilities.yml` then `update_docker_image.yml` and observe:
   - vulnerabilities report,
   - pre-change backup,
   - healthcheck gate,
   - rollback on failure.

2. **CI/CD gates**: open `.gitlab-ci.yml` and check:
   - build/test/deploy stages,
   - failure blocking behavior,
   - deployment via Ansible with verification.

3. **Audit trail**: see how WeKan tickets are created/updated for traceability.

## Projects

### [Docker Vulnerability Automation](./docker-vulnerability-automation/)

Ansible-based automation for Docker image vulnerability lifecycle management.

**Key Features:**
- Automated CVE scanning with Trivy
- Docker image updates with automatic rollback on failure
- HTTP healthcheck verification before completion
- WeKan integration for change tracking and audit compliance
- Ansible Vault for secure credential management

**Workflow:**
```
Scan (Trivy) → Backup → Update → Pull → Restart → Healthcheck → Verify
                  ↓                                      ↓
              Rollback ←←←←←←←←←← (on failure) ←←←←←←←←←┘
```

### [GitLab CI/CD Pipeline](./gitlab-cicd-pipeline/)

Complete CI/CD implementation for automated application deployment.

**Key Features:**
- 3-stage pipeline: Build → Test → Deploy
- Custom GitLab Runner with Docker-in-Docker support
- Automated syntax validation and functional testing
- Ansible-based deployment with healthcheck verification
- Encrypted inventory with Ansible Vault

**Pipeline Flow:**
```
  git push
      │
      ▼
┌──────────┐     ┌──────────┐     ┌──────────┐
│  BUILD   │ ──▶ │   TEST   │ ──▶ │  DEPLOY  │
│  docker  │     │  lint    │     │  ansible │
│  build   │     │  curl    │     │  health  │
└──────────┘     └──────────┘     └──────────┘
                       │
                 (blocks on failure)
```

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Infrastructure Security Stack                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌────────────┐     ┌────────────┐     ┌────────────────────────┐  │
│   │   GitLab   │────▶│   Runner   │────▶│     Target Server      │  │
│   │   Server   │     │  (Docker + │     │   ┌────────────────┐   │  │
│   │            │     │   Ansible) │     │   │  Application   │   │  │
│   └────────────┘     └────────────┘     │   │  (Container)   │   │  │
│                            │            │   └────────────────┘   │  │
│                            │            │           │            │  │
│                            ▼            │           ▼            │  │
│                     ┌────────────┐      │   ┌────────────────┐   │  │
│                     │   Trivy    │      │   │  Healthcheck   │   │  │
│                     │  Scanner   │      │   │  Monitoring    │   │  │
│                     └────────────┘      │   └────────────────┘   │  │
│                            │            │                        │  │
│                            ▼            └────────────────────────┘  │
│                     ┌────────────┐                                  │
│                     │   WeKan    │                                  │
│                     │  Tracking  │                                  │
│                     └────────────┘                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Technical Stack

| Category | Technologies |
|----------|--------------|
| **Automation** | Ansible, Docker, Docker Compose |
| **CI/CD** | GitLab CI, Custom Runner, Pipeline Stages |
| **Security** | Trivy CVE Scanner, Ansible Vault, Secrets Management |
| **Reliability** | Healthchecks, Automatic Rollback, Block/Rescue Patterns |
| **Traceability** | WeKan API, Automated Ticketing |

## Control Mapping (high-level)

| Control Area | Implementation |
|--------------|----------------|
| **Supply chain / container hygiene** | CVE scanning (Trivy), controlled rollout, rollback |
| **NIST SSDF** | Automated checks in CI, deployment verification, traceability |
| **SLSA-minded practices** | Pipeline gates, repeatable deployments (IaC) |
| **Change management** | Ticket-based audit trail (WeKan) |

## Quick Start

### Vulnerability Scanning

```bash
cd docker-vulnerability-automation/ansible

# Scan current image for vulnerabilities
ansible-playbook scan_vulnerabilities.yml

# Update image with automatic rollback protection
ansible-playbook update_docker_image.yml \
  -e "image_name=nginx image_version=1.24.0"
```

### CI/CD Pipeline

```bash
cd gitlab-cicd-pipeline

# Build and register custom runner
cd runner
DOCKER_GID=$(getent group docker | cut -d: -f3)
docker build --build-arg DOCKER_GID=$DOCKER_GID -t gitlab-runner-custom .

# Deploy application (triggered automatically on git push)
git add . && git commit -m "Deploy update" && git push
```

## Safety Features

| Feature | Description |
|---------|-------------|
| **Pre-update Backup** | Configuration saved before any modification |
| **HTTP Healthcheck** | Application verified responsive (5 retries, 3s delay) |
| **Automatic Rollback** | Previous version restored on any failure |
| **Test Gate** | Deployment blocked if syntax or functional tests fail |
| **Audit Trail** | Every action logged to ticketing system |

## Security Considerations

- All credentials encrypted with Ansible Vault
- No secrets stored in repository
- Example files provided for configuration templates
- SSH key-based authentication for deployments
- HTTPS recommended for production WeKan/GitLab endpoints

## Hardening Notes (what I'd do in real production)

- Run Trivy as a CI job + enforce severity thresholds (policy-as-code)
- Sign images (cosign) and verify signatures at deploy time
- Protect runners (isolated, least privilege, no Docker socket where possible)
- Centralize logs/metrics (Prometheus/Grafana) and alert on rollback events
- Store secrets in a dedicated vault (e.g., HashiCorp Vault) + short-lived credentials
- Add SBOM generation (Syft) and store artifacts for audit

## Requirements

- Docker & Docker Compose
- Ansible (tested with modern releases)
- GitLab (for CI/CD pipeline)
- Python 3.x
- Trivy (runs as container)

## Project Structure

```
infrastructure-security-automation/
├── docker-vulnerability-automation/
│   ├── ansible/
│   │   ├── scan_vulnerabilities.yml
│   │   ├── update_docker_image.yml
│   │   ├── update_docker_image_wekan.yml
│   │   └── vault.yml.example
│   ├── application/
│   │   └── docker-compose.yml
│   └── README.md
│
├── gitlab-cicd-pipeline/
│   ├── src/
│   │   └── index.php
│   ├── runner/
│   │   └── Dockerfile
│   ├── Dockerfile
│   ├── .gitlab-ci.yml
│   ├── playbook.yml
│   └── README.md
│
├── LICENSE
├── CONTRIBUTING.md
└── README.md
```

## Author

**Laurent Giovannoni**

Focus: industrializing security controls in CI/CD and production operations.

## License

MIT License - See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
