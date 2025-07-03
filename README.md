# Mointering GitOps Repository

This repository contains the GitOps configuration for deploying the Mointering application using ArgoCD.

## Overview

This repository follows the GitOps pattern using ArgoCD to manage the deployment of the Mointering application stack, which includes:

- Frontend service
- Backend services (HTTP API and Worker)
- Redis cache
- RabbitMQ message broker

## Repository Structure

```
.
├── apps/
│   ├── UI/
│   │   └── frontend.yaml    # Frontend application definition
│   ├── backend/
│   │   ├── http-backend.yaml    # HTTP API service
│   │   ├── worker-backend.yaml  # Background worker service
│   │   ├── redis.yaml          # Redis cache
│   │   └── rabbit.yaml         # RabbitMQ message broker
│   └── roots.yaml             # Root application that discovers other apps
```

## Prerequisites

- Kubernetes cluster with ArgoCD installed
- `kubectl` configured to communicate with your cluster
- ArgoCD CLI (optional but recommended)

## Getting Started

### 1. Bootstrap ArgoCD

If you haven't already installed ArgoCD in your cluster:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Deploy Applications

The root application will automatically discover and deploy all other applications:

```bash
kubectl apply -f apps/roots.yaml
```

### 3. Access ArgoCD UI

To access the ArgoCD web interface:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Then open https://localhost:8080 in your browser.

## Application Details

### Frontend

- **Namespace**: `frontend`
- **Image**: `ring0exec/frontend:latest`
- **Source**: [GitHub - mointering/frontend](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/frontend)

### Backend Services

#### HTTP Backend
- **Namespace**: `backend`
- **Image**: `ring0exec/http-backend:latest`
- **Source**: [GitHub - mointering/http-backend](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/http-backend)

#### Worker Backend
- **Namespace**: `backend`
- **Image**: `ring0exec/worker-backend:latest`
- **Source**: [GitHub - mointering/backend](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/backend)

### Dependencies

#### Redis
- **Namespace**: `backend`
- **Source**: [GitHub - mointering/redis](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/redis)

#### RabbitMQ
- **Namespace**: `backend`
- **Source**: [GitHub - mointering/rabbitmq](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/rabbitmq)

## CI/CD Pipeline & Image Management

### Image Tag Strategy
- The `image.tag: latest` in the this repository is a placeholder
- During the CI/CD pipeline execution, this is automatically replaced with the Git commit hash (e.g., `bdjfhdjghf876ry43gfb8847`)
- This ensures every deployment is traceable to a specific code version

### Deployment Flow
1. Code is pushed to the mointering repository
2. CI/CD pipeline is triggered
3. Application is built and containerized
4. Image is tagged with the Git commit hash
5. Image is pushed to the container registry
6. main repo clone this repository and make commit with the new image tag
7. ArgoCD detects the change and deploys the new version

### Sync Policy

All applications are configured with the following sync policy:
- **Automated Sync**: Enabled
- **Prune**: Enabled (removes resources when removed from Git)
- **Self Heal**: Enabled (corrects any manual changes to match Git state)

## Verifying Deployments

To check which image version is currently deployed:

```bash
# List all deployments and their images
kubectl get deployments --all-namespaces -o jsonpath="{range .items[*]}{'\n'}{.metadata.namespace}{'/'}{.metadata.name}{': '}{range .spec.template.spec.containers[*]}{.image}{', '}{end}{end}"

# Check a specific deployment
kubectl get deployment -n <namespace> <deployment-name> -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Troubleshooting

### View Application Status
```bash
argocd app list
```

### Check Application Logs
```bash
argocd app logs <app-name>
```

### Sync Application Manually
```bash
argocd app sync <app-name>
```
