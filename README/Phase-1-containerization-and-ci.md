# Phase 1 — Containerization & Continuous Integration (CI)

## Purpose of This Phase

The goal of Phase 1 is to answer one simple question:

> **How do we reliably turn application code into a Docker image, automatically, every time code is pushed?**

By the end of this phase:

* The application runs inside a Docker container
* GitHub Actions automatically builds the app
* The Docker image is pushed to Docker Hub
* Every build is traceable and reproducible

No Kubernetes yet. No AWS yet.
This phase focuses **only on Docker and CI**.

---

## Understanding the Application (High Level)

The application is a **Next.js web application**.

Important points:

* Runs on **Node.js**
* Exposes the app on **port 3000**
* Built using `npm`
* No database dependency in this phase

We treat the app as a **black box**:

* If `npm run build` works, the app is considered valid
* If it fails, the build must stop

---

## Why Containerization Is Needed

Without Docker:

* Code runs differently on different machines
* CI servers may behave differently than laptops
* Deployments become unpredictable

With Docker:

* The same container runs everywhere
* CI, local, and cloud environments are consistent
* Kubernetes can run the app later without changes

So the **first technical task** is to containerize the app.

---

## Writing the Dockerfile

The Dockerfile defines **how the application is built and run**.

This project uses a **multi-stage Docker build** to keep images smaller and cleaner.

### Dockerfile Used

```dockerfile
# ---------- deps ----------
FROM node:20-alpine AS deps
WORKDIR /app

COPY package.json ./
RUN npm install --legacy-peer-deps

# ---------- builder ----------
FROM node:20-alpine AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

# ---------- runner ----------
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app/package.json ./
COPY --from=builder /app/next.config.mjs ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000

CMD ["npm", "start"]
```

---

## What Each Stage Does (Simple Explanation)

### 1. `deps` stage

* Installs dependencies using `npm install`
* Cached to avoid reinstalling every time

### 2. `builder` stage

* Copies application source code
* Runs `npm run build`
* Fails if the app cannot be built

### 3. `runner` stage

* Contains only what is needed to run the app
* Starts the application using `npm start`
* Exposes port `3000`

---

## Why Multi-Stage Builds Are Used

* Keeps the final image smaller
* Separates build-time and run-time concerns
* Common industry practice

---

## Docker Ignore File

A `.dockerignore` file is added to avoid copying unnecessary files into the image.

### `.dockerignore`

```
node_modules
.git
.github
k8s
README.md
npm-debug.log
```

This:

* Speeds up builds
* Reduces image size
* Avoids leaking repo metadata

---

## Image Tagging Strategy

Images are **not** tagged as `latest`.

Instead, each image is tagged using:

* **Git commit SHA**

Example:

```
vkuldeep1/innovibe-site:7f63a4727450591e76079a2f5f0a20dbf10e3f2a
```

Why this matters:

* Every image is immutable
* Easy rollback
* Exact traceability from image → code commit

---

## Introducing Continuous Integration (CI)

CI ensures:

* Every code push is validated
* Broken code never produces a Docker image
* Builds are automatic and consistent

This project uses **GitHub Actions** for CI.

---

## CI Workflow Overview

The CI pipeline performs these steps:

1. Checkout source code
2. Install dependencies
3. Build the application
4. Build Docker image
5. Push image to Docker Hub

If **any step fails**, the pipeline stops.

---

## GitHub Actions Workflow

File location:

```
.github/workflows/ci.yml
```

### CI Configuration

```yaml
name: CI - Build, Test, Push

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install --legacy-peer-deps

      - name: Build application
        run: npm run build

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            vkuldeep1/innovibe-site:${{ github.sha }}
```

---

## Why `npm run build` Is Considered a Test

This project does not yet include unit tests.

Instead:

* If `npm run build` fails
* The application is considered broken
* The Docker image is not pushed

This is a **minimum acceptable CI gate**.

---

## GitHub Secrets Used

The following secrets are stored in GitHub:

* `DOCKERHUB_USERNAME`
* `DOCKERHUB_TOKEN`

They are:

* Not committed to code
* Only accessible to GitHub Actions
* Required for pushing images securely

---

## Common Issues Faced in This Phase

### 1. Dependency lock confusion

* The project originally contained a `bun.lock`
* The app used `npm`, not Bun
* This caused Docker build failures
* Solution: standardize on one package manager

### 2. Missing `.dockerignore`

* Caused large build contexts
* Fixed by adding `.dockerignore`

### 3. CI syntax errors

* Minor YAML mistakes broke pipelines
* Fixed by validating workflow structure

These issues were resolved during implementation and documented intentionally.

---

## Outcome of Phase 1

At the end of this phase:

✅ Application is containerized
✅ Docker image builds reliably
✅ CI runs automatically on every push
✅ Images are pushed to Docker Hub
✅ Each image is traceable to a commit

---

## What Phase 1 Does NOT Do

* No Kubernetes
* No AWS
* No deployment
* No public access

Those are handled in later phases.

---

## Next Phase

➡️ **Phase 2 — Kubernetes Manifests & Deployment Basics**

This will explain:

* Kubernetes Deployment
* Service configuration
* Resource limits
* Image usage strategy

---

