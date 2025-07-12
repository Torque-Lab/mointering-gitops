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
├── argocd/
│   ├── frontend.yaml        # ArgoCD Application for the frontend Helm chart
│   ├── http-backend.yaml    # ArgoCD Application for the HTTP API service
│   ├── worker-backend.yaml  # ArgoCD Application for the background worker
│   ├── redis.yaml           # ArgoCD Application for Redis
│   ├── rabbit.yaml          # ArgoCD Application for RabbitMQ
│   └── ingress.yaml         # ArgoCD Application for ingress rules
├── apps/
│   └── charts/              # In-house Helm charts
│       ├── frontend/
│       ├── global/
│       ├── http-backend/
│       ├── worker-backend/
│       ├── redis/
│       ├── rabbitmq/
│       └── ingress/
├── mointering-root.yaml     # Root ArgoCD Application that bootstraps all of the above
└── README.md
```

### Some decisions why made so
1. You will see  that  we have two ingress one for frontend and one for backend and two sealed secrets one for frontend and one for backend, you will be wondering why we have two ingress and two sealed secrets, the reason is that  ingress can't route traffic to different namespace as our backend is in backend namespace and frontend is in frontend namespace, and sealed secrets can't be used shared b/t namespaces, so we have to create two ingress and two sealed secrets one for frontend and one for backend
i.e moral of the story ingress can't reach service in different namespace and k8s not allow sharing sealed secrets b/t namespaces normally
2. our goal is to separate frontend and backend into different namespaces for better security and isolation
3. we have to create two sealed secrets one for frontend and one for backend

### some fun facts
1. frontend is in nextjs and backend is in express nodejs and they still available by same domain without causing any requirement of CORS
e.g  frontend is in https://sitewatch.suvidhaportal.com and backend is in https://sitewatch.suvidhaportal.com/api/, and note no any security, i repeat no means no security issue and this mimic like nextjs backend system being in same domain and still separated
3. you will be wondering why this whole drama because i hate nextjs as backend system so i found solution to make it work in same domain,my sytem will totally behave like they are couple together but still separated
4. more details visit source repo https://github.com/Torque-Lab/mointering
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
kubectl apply -f mointering-root.yaml
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
- **Image**: `ring0exec/frontend:tag`
- **Source**: [GitHub - mointering/frontend](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/frontend)

### Backend Services

#### HTTP Backend
- **Namespace**: `backend`
- **Image**: `ring0exec/http-backend:tag`
- **Source**: [GitHub - mointering/http-backend](https://github.com/Torque-Lab/mointering/tree/main/mointering/charts/http-backend)

#### Worker Backend
- **Namespace**: `backend`
- **Image**: `ring0exec/worker-backend:tag`
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

