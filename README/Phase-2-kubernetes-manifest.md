# Phase 2 — Kubernetes Manifests & Deployment Basics

## Purpose of This Phase

The goal of Phase 2 is to answer this question:

> **Once we have a Docker image, how do we run it in Kubernetes in a controlled and repeatable way?**

In this phase, we:

* Define how the application runs inside Kubernetes
* Control resources (CPU and memory)
* Decide how traffic reaches the application
* Prepare the app to be deployed on EKS later

No CI/CD changes yet.
No AWS infrastructure yet.
This phase is **pure Kubernetes basics**.

---

## Why Kubernetes Is Needed

Docker alone can run containers, but it does not handle:

* Restarts when a container crashes
* Scaling replicas
* Networking between containers
* Rolling updates

Kubernetes solves these problems by:

* Managing containers as Pods
* Restarting failed containers
* Exposing applications via Services
* Allowing zero-downtime updates

---

## Kubernetes Files in This Project

All Kubernetes-related files are stored in one place:

```
k8s/
├── deployment.yaml
└── service.yaml
```

This keeps infrastructure concerns separate from application code.

---

## Kubernetes Deployment

A **Deployment** defines:

* Which image to run
* How many replicas to run
* How Kubernetes should update the app

---

### `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: innovibe-site
spec:
  replicas: 1
  selector:
    matchLabels:
      app: innovibe-site
  template:
    metadata:
      labels:
        app: innovibe-site
    spec:
      containers:
        - name: innovibe-site
          image: vkuldeep1/innovibe-site:<IMAGE_TAG>
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "64Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

---

## Explanation (Line by Line)

### `replicas: 1`

* Only one pod runs
* Chosen to minimize cost
* Enough for demo and testing

---

### `image`

* Uses Docker Hub image
* Tagged using Git commit SHA
* Ensures Kubernetes always runs a known version

---

### `containerPort: 3000`

* Matches the port exposed in the Dockerfile
* Kubernetes uses this for networking

---

### Resource Requests & Limits

```yaml
requests:
  memory: "64Mi"
  cpu: "100m"
limits:
  memory: "256Mi"
  cpu: "500m"
```

Why this matters:

* Prevents the app from consuming too many resources
* Protects the cluster
* Required in real-world clusters

---

## Kubernetes Service

A **Service** exposes the application to the network.

Without a Service:

* Pods are not reachable
* IPs change constantly

---

### `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: innovibe-site
spec:
  type: LoadBalancer
  selector:
    app: innovibe-site
  ports:
    - port: 80
      targetPort: 3000
```

---

## Why `type: LoadBalancer` Was Chosen

Kubernetes supports multiple Service types:

| Type         | Purpose                          |
| ------------ | -------------------------------- |
| ClusterIP    | Internal only                    |
| NodePort     | Exposes node ports (not ideal)   |
| LoadBalancer | Public access via cloud provider |

This project requires:

* A **public IP**
* Cloud-managed networking

So `LoadBalancer` is the correct choice.

On AWS:

* This automatically creates an AWS Load Balancer
* Returns a public DNS name

---

## How Deployment and Service Work Together

1. Deployment creates Pods
2. Pods are labeled `app=innovibe-site`
3. Service selects Pods using the label
4. Traffic is routed to the Pods automatically

No hardcoded IPs are used.

---

## Applying Kubernetes Manifests

Manifests are applied using:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Verification commands:

```bash
kubectl get pods
kubectl get svc
```

---

## Common Issues Encountered in This Phase

### 1. Using NodePort initially

* Did not provide a real public endpoint
* Replaced with `LoadBalancer`

### 2. No pods running

* Service created before Deployment
* Fixed by applying Deployment first

### 3. Image pull failures in local clusters

* Caused by local networking issues
* Not an architectural problem
* Resolved by moving to EKS

---

## What Phase 2 Does NOT Solve

* No cluster creation
* No AWS permissions
* No CI/CD integration
* No automatic deployment

These are intentionally deferred.

---

## Outcome of Phase 2

At the end of this phase:

✅ Kubernetes manifests are defined
✅ Application can run in Kubernetes
✅ Resource usage is controlled
✅ Public exposure is planned
✅ Manifests are cloud-ready

---

## Why This Phase Matters

This phase creates a **clear contract**:

> “Given an image, Kubernetes knows exactly how to run it.”

Everything after this builds on that contract.

---

## Next Phase

➡️ **Phase 3 — AWS EKS Infrastructure & Cluster Setup**

Phase 3 will cover:

* Creating the EKS cluster
* Node groups and cost control
* Kubernetes authentication
* Making the Service publicly accessible

---


