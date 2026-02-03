# Phase 3 — AWS EKS Infrastructure & Cluster Setup

## Purpose of This Phase

The goal of Phase 3 is to answer this question:

> **How do we run our Kubernetes manifests on real cloud infrastructure and make the application accessible from the internet?**

In this phase, we:

* Create an actual Kubernetes cluster on AWS
* Run our existing Kubernetes manifests without changes
* Configure authentication properly
* Expose the application publicly

This is where the project moves from **local experiments** to **real cloud deployment**.

---

## Why Amazon EKS

Amazon EKS (Elastic Kubernetes Service) is AWS’s managed Kubernetes offering.

Using EKS means:

* AWS manages the Kubernetes control plane
* We only manage worker nodes and workloads
* High availability is handled by AWS

This is the most common Kubernetes choice in AWS-based companies.

---

## Cost-Aware Design Decisions

This project was built using an AWS account with **credits / free-tier awareness**.

To minimize cost:

* Single cluster
* Single node
* Small instance size
* No unnecessary add-ons
* Cluster deleted after testing

These choices are **intentional**, not limitations.

---

## Tools Used in This Phase

* **AWS CLI** – to authenticate and manage AWS
* **eksctl** – to create and manage EKS clusters
* **kubectl** – to interact with Kubernetes

---

## Prerequisites Verified

Before creating the cluster, the following were confirmed:

```bash
aws sts get-caller-identity
eksctl version
kubectl version --client
```

This ensures:

* AWS credentials are valid
* Required tools are installed
* Kubernetes commands will work

---

## Creating the EKS Cluster

The cluster is created using `eksctl`, which internally uses CloudFormation.

### Cluster Creation Command

```bash
eksctl create cluster \
  --name innovibe-cluster \
  --region ap-south-1 \
  --nodegroup-name ng-1 \
  --node-type t3.small \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --managed
```

---

## Explanation of Key Options

### `--node-type t3.small`

* Keeps EC2 cost low
* Sufficient for a single web app

### `--nodes 1`

* One worker node
* Enough for demonstration
* Minimizes cost

### `--managed`

* Uses AWS-managed node groups
* Safer and simpler than self-managed nodes

---

## What eksctl Creates Automatically

This command creates:

* EKS control plane
* VPC with public and private subnets
* Managed node group
* IAM roles for nodes
* Kubernetes add-ons (CoreDNS, kube-proxy, VPC CNI)

All via CloudFormation.

---

## Verifying Cluster Creation

After creation:

```bash
kubectl config current-context
kubectl get nodes
```

Expected result:

* Context points to `innovibe-cluster`
* Node status is `Ready`

This confirms the cluster is usable.

---

## Deploying the Application to EKS

At this point, **no changes** are required to Kubernetes manifests.

The same files from Phase 2 are reused.

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

---

## Verifying Application Status

```bash
kubectl get pods
kubectl get svc
```

Expected:

* Pod status: `Running`
* Service type: `LoadBalancer`
* `EXTERNAL-IP` populated with AWS DNS name

---

## Public Access via AWS Load Balancer

Because `type: LoadBalancer` is used:

* AWS provisions a Load Balancer automatically
* A public DNS name is assigned
* The app becomes accessible from the internet

Example:

```
http://<aws-load-balancer-dns>
```

This confirms the app is running **on AWS**, not locally.

---

## Kubernetes Authentication Model (Important)

EKS uses **two layers of authentication**:

### 1. AWS IAM

* Controls who can call AWS APIs

### 2. Kubernetes RBAC

* Controls who can run `kubectl` commands

Only the cluster creator is trusted by default.

---

## The aws-auth ConfigMap

The `aws-auth` ConfigMap connects IAM identities to Kubernetes RBAC.

* Worker nodes are mapped using `mapRoles`
* CI users must be mapped using `mapUsers`

This mapping is required for CI/CD later.

---

## Common Issues Encountered

### 1. ImagePullBackOff (local clusters)

* Caused by local networking
* Not an issue on EKS

### 2. EXTERNAL-IP stuck in `<pending>`

* Happens in local Kubernetes
* Works correctly on AWS

### 3. Unauthorized kubectl access

* Caused by missing RBAC mappings
* Fixed by updating `aws-auth`

---

## What Phase 3 Does NOT Include

* No CI/CD deployment automation
* No ingress or HTTPS
* No autoscaling
* No database setup

Those are covered in later phases or intentionally excluded.

---

## Outcome of Phase 3

At the end of this phase:

✅ Real EKS cluster is running
✅ Kubernetes manifests work without changes
✅ Application is publicly accessible
✅ AWS networking is correctly configured
✅ Foundation for CD is ready

---

## Why This Phase Matters

This phase proves that:

* The app is cloud-ready
* Kubernetes manifests are portable
* Infrastructure and application are cleanly separated

---

## Next Phase

➡️ **Phase 4 — Continuous Deployment (CI → EKS Automation)**

Phase 4 will explain:

* How GitHub Actions deploys to EKS
* IAM + RBAC integration
* Automated rolling updates
* End-to-end pipeline behavior

---


