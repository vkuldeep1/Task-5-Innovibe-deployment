# 📄 `overview(readme.md)`

## Project Overview — End-to-End CI/CD with Docker, GitHub Actions, and AWS EKS

This repository demonstrates a **complete production-style DevOps workflow** for deploying a modern web application using containers, CI/CD, and Kubernetes on AWS.

The goal of this project is **not just deployment**, but to show how different DevOps concepts work **together as a single system**, from source code to a publicly accessible application.

---

## What This Project Does

At a high level, this project implements the following flow:

1. Application source code is pushed to GitHub
2. A CI pipeline builds and validates the application
3. A Docker image is created and pushed to Docker Hub
4. A CD pipeline automatically deploys the new image to AWS EKS
5. The application becomes accessible via a public AWS Load Balancer

All of this happens **automatically** after a `git push`.

---

## Core Technologies Used

* **GitHub** – Source code management
* **GitHub Actions** – CI/CD automation
* **Docker** – Application containerization
* **Docker Hub** – Container image registry
* **Kubernetes** – Container orchestration
* **Amazon EKS** – Managed Kubernetes on AWS
* **AWS Load Balancer** – Public access to the application

The database layer (RDS) is **intentionally excluded** from this project, as it is managed externally.

---

## Why This Architecture

This architecture reflects how real-world teams deploy applications:

* Developers only interact with **Git**
* CI systems handle **builds and validation**
* Containers provide **consistency across environments**
* Kubernetes manages **runtime, scaling, and restarts**
* Cloud infrastructure provides **reliability and public access**

The project avoids unnecessary complexity while still following **industry-accepted practices**.

---

## Repository Structure (High Level)

```
.
├── app/                     # Application source code
├── Dockerfile               # Container build definition
├── k8s/
│   ├── deployment.yaml      # Kubernetes Deployment
│   └── service.yaml         # Kubernetes Service (LoadBalancer)
├── .github/
│   └── workflows/
│       └── ci.yml           # CI/CD pipeline
├── overview.md              # High-level project explanation
└── README/                  # Detailed phase-by-phase documentation
```

---

## Documentation Philosophy

This repository uses **two levels of documentation**:

### 1. `overview.md` (this file)

* Explains **what the system is**
* Explains **how the pieces fit together**
* No deep commands or troubleshooting

### 2. `README/` folder (detailed)

For readers who want deeper understanding, the work is broken into **four clear phases**, each explained in simple language with:

* Commands used
* Decisions made
* Errors faced
* Fixes applied
* Reasoning behind choices

---

## Documentation Phases

The detailed documentation is split into **four phases only** (intentionally limited):

### Phase 1 — Containerization & CI

* Understanding the application
* Writing the Dockerfile
* Setting up GitHub Actions
* Building, testing, and pushing images

### Phase 2 — Kubernetes Manifests

* Kubernetes Deployment and Service
* Resource limits
* Image tagging strategy
* Local testing considerations

### Phase 3 — AWS EKS Infrastructure

* Creating the EKS cluster
* Node groups and cost-aware choices
* Kubernetes authentication (IAM + RBAC)
* Exposing the application publicly

### Phase 4 — Continuous Deployment (CD)

* Connecting GitHub Actions to EKS
* Handling authentication (`aws-auth`)
* Automated rolling updates
* End-to-end validation

Each phase is documented **independently**, so readers can jump directly to what they care about.

---

## Intended Audience

This project is useful for:

* DevOps beginners learning real workflows
* Students preparing for DevOps/Cloud interviews
* Engineers wanting a reference EKS CI/CD setup
* Anyone who wants to understand *how everything connects*

---

## Scope and Limitations

Included:

* CI/CD
* Docker
* Kubernetes
* AWS EKS
* Public access

Excluded (by design):

* Database provisioning
* Secrets management beyond basics
* Advanced Kubernetes patterns (Ingress, HPA, GitOps)

These can be added later without changing the core design.

---

## Summary

This project shows how to:

* Build once
* Ship consistently
* Deploy automatically
* Run on real cloud infrastructure

The rest of the documentation explains **how each part was built**, step by step.

---

## Next

➡️ Continue to:
**`README/phase-1-containerization-and-ci.md`**

---


