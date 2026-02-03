# Phase 4 — Continuous Deployment (CI → EKS Automation)

## Purpose of This Phase

The goal of Phase 4 is to answer the final question:

> **How does a code push automatically update the running application on EKS without manual steps?**

This phase completes the CI/CD pipeline by adding **Continuous Deployment (CD)**.

After this phase:

* A `git push` triggers a full deployment
* No manual `kubectl apply` is needed
* Kubernetes performs rolling updates automatically

---

## What We Already Have Before Phase 4

From previous phases, we already have:

* Docker images built and pushed on every commit
* Immutable image tags (Git commit SHA)
* A running EKS cluster
* Kubernetes Deployment and Service working
* Public access via AWS LoadBalancer

What is missing:

* Automatic deployment to EKS from CI

---

## CD Design Choice (Kept Simple)

There are many ways to do CD (Helm, ArgoCD, Flux, GitOps).

For this project, we intentionally choose:

### **kubectl-based deployment from GitHub Actions**

Why:

* Easy to understand
* Minimal moving parts
* Common in small-to-medium setups
* Easy to explain in interviews and reviews

This is **good enough and correct** for the project scope.

---

## High-Level CD Flow

```
git push
→ GitHub Actions builds image
→ Image pushed to Docker Hub
→ GitHub Actions authenticates to AWS
→ kubectl updates Deployment image
→ Kubernetes performs rolling update
```

---

## Authentication Problem to Solve

GitHub Actions must be able to:

1. Call AWS APIs
2. Run `kubectl` commands on the EKS cluster

These are **two different permissions**.

---

## Authentication Layers Explained (Simple)

### 1. AWS IAM (Cloud Access)

* Allows GitHub Actions to call `aws eks update-kubeconfig`
* Uses AWS access keys stored as GitHub secrets

### 2. Kubernetes RBAC (Cluster Access)

* Allows `kubectl` commands inside the cluster
* Managed via `aws-auth` ConfigMap

Both are required.

---

## IAM User for CI/CD

A dedicated IAM user was created:

```
github-actions-eks
```

Why not reuse a personal user:

* Separation of responsibility
* Easier rotation and auditing
* Safer for automation

---

## GitHub Secrets Used for CD

The following secrets are added to the GitHub repository:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
EKS_CLUSTER_NAME
```

These are:

* Encrypted
* Not visible in logs
* Only accessible to GitHub Actions

---

## Kubernetes RBAC Mapping (aws-auth)

EKS does **not** automatically trust IAM users.

The CI IAM user must be explicitly mapped.

### aws-auth Mapping Added

```yaml
mapUsers: |
  - userarn: arn:aws:iam::<ACCOUNT_ID>:user/github-actions-eks
    username: github-actions
    groups:
      - system:masters
```

This allows GitHub Actions to:

* Read cluster state
* Update Deployments
* Perform rollouts

> Note: `system:masters` is powerful, but acceptable for a demo/project.
> In production, this would be more restricted.

---

## Updating the CI Pipeline to Add CD

The existing CI workflow is extended.

### New Steps Added (After Image Push)

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ secrets.AWS_REGION }}

- name: Update kubeconfig
  run: |
    aws eks update-kubeconfig \
      --name ${{ secrets.EKS_CLUSTER_NAME }} \
      --region ${{ secrets.AWS_REGION }}

- name: Deploy to EKS
  run: |
    kubectl set image deployment/innovibe-site \
      innovibe-site=vkuldeep1/innovibe-site:${{ github.sha }}
```

---

## Why `kubectl set image` Is Used

Instead of editing YAML files, we:

* Update only the image field
* Trigger a rolling update
* Avoid committing deployment changes to git

Benefits:

* Clean separation of code and runtime state
* Simple rollback
* Less YAML duplication

---

## Rolling Update Behavior

When the image is updated:

* Kubernetes creates a new Pod
* Waits for it to become ready
* Terminates the old Pod

Even with a single replica:

* Downtime is minimal
* Behavior is predictable

---

## Verifying CD Works

To test CD:

1. Make a small code or file change
2. Commit and push to `main`
3. Observe GitHub Actions:

   * Build
   * Push
   * Deploy
4. Check rollout:

   ```bash
   kubectl rollout status deployment/innovibe-site
   ```
5. Refresh the public URL

If the change is visible, CD is working.

---

## Common Issues Encountered

### 1. “You must be logged in to the server”

* Cause: IAM user not mapped in `aws-auth`
* Fix: Add `mapUsers` entry

### 2. kubectl works locally but not in CI

* Cause: Missing RBAC permissions
* Fix: Verify `aws-auth` mapping

### 3. Image updates but app doesn’t change

* Cause: Same image tag reused
* Fix: Use SHA-based immutable tags (already implemented)

---

## Outcome of Phase 4

At the end of this phase:

✅ CI and CD are fully automated
✅ GitHub Actions deploys to EKS
✅ No manual kubectl commands needed
✅ Rolling updates work correctly
✅ End-to-end pipeline is complete

---

## Final System Summary

The final system behaves as follows:

```
Developer → GitHub → CI → Docker Hub → CD → EKS → Public URL
```

Each step is automated, reproducible, and auditable.

---

## What This Project Deliberately Does NOT Include

* GitOps tools (ArgoCD / Flux)
* HTTPS / Ingress
* Autoscaling (HPA)
* Advanced RBAC
* Database provisioning

These are optional improvements and not required for the core objective.

---

## Final Notes on Cost Control

EKS incurs cost even when idle.

After testing:

```bash
eksctl delete cluster --name innovibe-cluster --region ap-south-1
```

This prevents unnecessary credit usage.

---
