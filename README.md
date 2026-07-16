# GitHub Actions Self-Hosted Runner

## Overview

This project demonstrates how to execute GitHub Actions workflows using:

- A traditional Self-Hosted Runner
- A Kubernetes-based Self-Hosted Runner using Actions Runner Controller (ARC)

The objective is to understand the differences between both approaches, their architecture, scalability, and practical use cases.

---

# Project Architecture

## Traditional Self-Hosted Runner

```text
Developer
     │
 git push
     │
     ▼
GitHub Repository
     │
     ▼
GitHub Actions
     │
     ▼
Self-Hosted Runner
     │
     ▼
Execute Workflow
```

---

## Kubernetes + ARC

```text
Developer
     │
 git push
     │
     ▼
GitHub Repository
     │
     ▼
GitHub Actions
     │
     ▼
Actions Runner Controller (ARC)
     │
     ▼
Kubernetes Cluster
     │
     ▼
Runner Pod
     │
     ▼
Execute Workflow
```

---

# Part 1 - Traditional Self-Hosted Runner

## Objective

Run GitHub Actions workflows on a machine that you own instead of using GitHub-hosted runners.

---

## Environment

- Windows
- WSL Ubuntu
- GitHub Repository
- GitHub Actions

---

## Steps

### 1. Create a GitHub repository

Example:

```
argocd_test
```

---

### 2. Create a workflow

```
.github/workflows/test.yml
```

Example:

```yaml
name: Self Hosted Demo

on:
  push:

jobs:
  demo:

    runs-on: self-hosted

    steps:

      - uses: actions/checkout@v4

      - name: Hello

        run: echo "Hello from Self Hosted Runner"
```

---

### 3. Register a Self-Hosted Runner

Navigate to:

```
Repository

→ Settings

→ Actions

→ Runners

→ New Self-hosted Runner
```

Download the runner package.

Configure it.

Start it.

---

### 4. Test

```bash
git add .

git commit -m "Test"

git push
```

GitHub sends the workflow directly to the runner.

---

## Advantages

- Free
- Full control
- Easy to configure
- Suitable for small projects

---

## Limitations

- Dependencies must be installed manually.
- Limited scalability.
- Limited isolation.
- Difficult to manage many concurrent workflows.

---

# Part 2 - Kubernetes + ARC

## Objective

Automatically create GitHub runners inside Kubernetes Pods.

Instead of executing workflows directly on the server, Kubernetes dynamically creates Runner Pods.

---

# Environment

- Docker
- Minikube
- Kubernetes
- kubectl
- Helm
- Actions Runner Controller
- GitHub Personal Access Token

---

# Installation Steps

## 1. Install Docker

Docker is required to run Minikube.

---

## 2. Install Minikube

Create a local Kubernetes cluster.

```bash
minikube start
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
NAME        STATUS
minikube    Ready
```

---

## 3. Install kubectl

Install the Kubernetes CLI.

Verify:

```bash
kubectl version --client
```

---

## 4. Install Helm

Helm is the package manager for Kubernetes.

Verify:

```bash
helm version
```

---

## 5. Install cert-manager

Required by ARC.

---

## 6. Install Actions Runner Controller

```bash
helm install ...
```

ARC becomes a Kubernetes Operator responsible for managing GitHub runners.

---

## 7. Create the GitHub Secret

Store the GitHub Personal Access Token.

```bash
kubectl create secret generic controller-manager \
    -n arc-system \
    --from-literal=github_token=<TOKEN>
```

---

## 8. Create RunnerDeployment

runner.yaml

```yaml
apiVersion: actions.summerwind.dev/v1alpha1

kind: RunnerDeployment

metadata:

  name: github-runner

spec:

  replicas: 1

  template:

    spec:

      repository: tahaelidrissi/argocd_test
```

Deploy:

```bash
kubectl apply -f runner.yaml
```

---

## What happens internally?

```
RunnerDeployment
        │
        ▼
Actions Runner Controller
        │
        ▼
Create Runner
        │
        ▼
Create Kubernetes Pod
        │
        ▼
Register Runner to GitHub
```

No Pod is created manually.

Everything is managed by ARC.

---

## 9. Execute a Workflow

```bash
git push
```

GitHub detects the workflow.

ARC receives the job.

Kubernetes creates a Runner Pod.

The Runner Pod executes the workflow.

Once completed, the Pod is destroyed.

---

# Runner Lifecycle

```text
Git Push
    │
    ▼
GitHub Actions
    │
    ▼
ARC detects a job
    │
    ▼
Create Runner Pod
    │
    ▼
Execute Workflow
    │
    ▼
Workflow Completed
    │
    ▼
Delete Runner Pod
```

---

# Comparison

| Traditional Runner | Kubernetes + ARC |
|--------------------|------------------|
| Fixed Runner | Dynamic Runner Pods |
| Installed directly on the server | Created on demand |
| Manual dependency management | Custom Runner Image |
| Limited scalability | Automatic scaling |
| Shared execution environment | Isolated execution |
| Runner always running | Ephemeral runners |

---

# Key Concepts Learned

- GitHub Actions
- Self-Hosted Runner
- Docker
- Kubernetes
- Minikube
- kubectl
- Helm
- Actions Runner Controller (ARC)
- RunnerDeployment
- Kubernetes Pods
- Kubernetes Secrets
- GitHub Personal Access Token

---

# Lessons Learned

During this lab, I learned how GitHub Actions integrates with Kubernetes through Actions Runner Controller.

Instead of running workflows directly on a server, ARC dynamically creates Kubernetes Pods acting as GitHub runners.

This architecture improves scalability, isolation, and resource utilization while providing a cloud-native approach to CI/CD.

---

# Technologies

- GitHub Actions
- Self-Hosted Runner
- Docker
- Kubernetes
- Minikube
- Helm
- Actions Runner Controller
- YAML
- WSL Ubuntu
