Below is the **same report rewritten in simple, clear language**, without jargon overload.
It explains **what you did, why you did it, and what you proved**, in a way **you can comfortably explain back**.

You can paste this directly into `README.md`.

---

# Innovibe Site – DevOps Work Report

## What this project is about

In this task, I took an existing web application and built a **complete DevOps flow** around it.
The goal was to:

* Run the app inside Docker
* Automatically build it using CI
* Deploy it on Kubernetes
* Measure memory and CPU usage
* Follow correct security and Git practices

This was done step by step, without shortcuts.

---

## Application Overview

* The app is a **Next.js frontend application**
* It runs on **Node.js**
* It listens on **port 3000**
* It is not a static website — it runs as a server

---

## Step 1: Getting the source code (SSH)

### Why SSH was used

GitHub no longer allows password login for Git.
SSH is the correct and secure way.

### What I did

* Created an SSH key on my system
* Added the public key to GitHub
* Tested SSH connection
* Cloned the repository using SSH

This allowed me to work with Git securely without passwords.

---

## Step 2: Dockerizing the application

### Why Docker

Docker makes the app:

* Easy to run anywhere
* Consistent across machines
* Ready for CI and Kubernetes

### How it was done

* Used a **multi-stage Docker build**
* First stage installs dependencies
* Second stage builds the app
* Final stage runs only what is needed

This keeps the runtime clean and predictable.

### Important note

Some dependencies were not fully compatible with the latest React version, so:

* `npm install --legacy-peer-deps` was used
* This is common in real projects

---

## Step 3: Running and testing locally

After building the Docker image:

* The container was started locally
* The app was opened in the browser
* Logs were checked

### Local resource usage

Using `docker stats`:

* Memory usage was around **65–70 MB**
* CPU usage was near **0%** when idle

This showed the app was running normally.

---

## Step 4: Pushing the image to Docker Hub

### Why this is needed

Kubernetes and CI need a place to pull the image from.

### What I did

* Logged in to Docker Hub
* Tagged the image correctly
* Pushed it to the registry

The image was now available publicly.

---

## Step 5: CI using GitHub Actions

### What CI does

Every time code is pushed:

* The Docker image is built automatically
* The image is pushed to Docker Hub
* The image is tagged using the commit ID

### Security

* Docker credentials were stored in GitHub Secrets
* No passwords or tokens were committed to the repo

CI ran successfully in about **2 minutes**.

---

## Step 6: Deploying to Kubernetes

### Kubernetes setup

* Used **Minikube** (local Kubernetes cluster)

### What was created

* **Deployment**

  * Runs the container
  * Controls replicas
* **Service**

  * Exposes the app using NodePort

### Resource limits

The container was limited so it can’t overuse system resources:

* Memory request: 64 MB
* Memory limit: 256 MB
* CPU request: 100m
* CPU limit: 500m

---

## Step 7: Verifying Kubernetes runtime

### Pod status

* Pod was running successfully
* No restarts
* Ready state was true

### Accessing the app

* Exposed using Minikube service
* App opened correctly in browser

### Resource usage (Kubernetes)

Using `kubectl top pod`:

* Memory usage: **~118 MB**
* CPU usage: **~1m**

This confirmed the app stayed within limits.

---

## Step 8: Git issue and fix 

### What went wrong

* A local tool (`minikube-linux-amd64`) was accidentally committed
* GitHub rejected the push due to file size limit

### How it was fixed

* The file was removed from Git history
* Git history was cleaned properly
* `.gitignore` was updated to prevent this in the future
* Changes were force-pushed safely

This fixed CI and kept the repo clean.

---

## Final Result

By the end of the task:

* The app runs inside Docker
* CI automatically builds and pushes images
* The app is deployed on Kubernetes
* Memory and CPU usage are measured and controlled
* Git repository is clean and secure

This is a **complete DevOps pipeline**, not just deployment.

---
