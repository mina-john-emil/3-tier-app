# 🚀 Mina Chat Tutorial — 3-Tier App on Kubernetes

> **Reactions & mood — how's the room?**  
> A full-stack, cloud-native 3-tier web application deployed on Kubernetes using a complete GitOps CI/CD pipeline with Azure DevOps and ArgoCD.



## 🌟 Overview

**Mina Chat Tutorial** is a production-grade 3-tier web application that demonstrates:

- A **React/HTML frontend** with dark-themed UI for real-time reactions and mood voting
- A **Node.js/Python backend** API with session-based authentication
- A **MySQL** relational database with **Redis** caching layer
- Full **containerization** with Docker
- **Kubernetes** orchestration with Helm charts
- **GitOps** deployment via ArgoCD (continuous sync from Git)
- **Azure DevOps** CI/CD pipeline for automated build & push

The app allows users to sign up, sign in, post emoji reactions to a shared **Reaction Wall**, and vote on the **Mood of the Day** — all persisted to MySQL, with recent data served from Redis cache for performance.

---

## 🏗️ Architecture

```
                         ┌─────────────────────────────────────────────┐
                         │              Kubernetes Cluster              │
                         │                                             │
  Internet ──► Ingress (buzzboard.local) ──► buzzboard-ingress-api    │
                         │                        │                    │
                         │                        ▼                    │
                         │              ┌─────────────────┐           │
                         │              │  Frontend (Pod)  │           │
                         │              │   mood-svc       │           │
                         │              │   reactions-svc  │           │
                         │              │   frontend-svc   │           │
                         │              └────────┬────────┘           │
                         │                       │                    │
                         │              ┌────────▼────────┐           │
                         │              │  Backend (API)   │           │
                         │              │   Node.js/Python │           │
                         │              └───┬─────────┬───┘           │
                         │                  │         │               │
                         │         ┌────────▼──┐  ┌───▼──────┐       │
                         │         │  MySQL DB  │  │  Redis   │       │
                         │         │  mysql-0   │  │  redis-0 │       │
                         │         └────────────┘  └──────────┘       │
                         └─────────────────────────────────────────────┘
```

**Tiers:**

| Tier | Component | Description |
|------|-----------|-------------|
| **Tier 1** | Frontend | Static/SPA served via Ingress, dark-themed UI |
| **Tier 2** | Backend API | Business logic, auth, session management |
| **Tier 3** | Data Layer | MySQL (persistent storage) + Redis (cache) |

---

## 🛠 Tech Stack

| Category | Technology |
|----------|-----------|
| **Frontend** | HTML5, CSS3, JavaScript (dark theme UI) |
| **Backend** | Node.js / Python |
| **Database** | MySQL 8 |
| **Cache** | Redis |
| **Containerization** | Docker, Docker Compose |
| **Orchestration** | Kubernetes (K8s) |
| **Package Manager** | Helm |
| **CI/CD** | Azure DevOps Pipelines |
| **GitOps** | ArgoCD v3.3.6 |
| **Registry** | Docker Hub / Azure Container Registry |
| **SSL/TLS** | Self-signed certs (TLS termination at Ingress) |
| **Version Control** | Azure Repos (Git) |

---

## ✨ Application Features

### 🔐 Authentication
- **Sign Up** — Create an account; username & password stored securely
- **Sign In** — Session-based authentication; reactions and mood votes saved per user
- **Log Out** — Session invalidation

### 😊 Mood of the Day
- Authenticated users vote on the room's mood:
  - 😵 Mostly surviving
  - 😀 It's okay
  - 🔥 On fire
- Live vote tallies displayed (e.g., `😵 2 · 😀 5 · 🔥 0`)
- Data source shown: `Last load: 2 ms · Source: MySQL` (or Redis when cached)
- Each vote is recorded with the user's account

### 💬 Reaction Wall
- Post short messages or emoji reactions to a shared wall
- Quick-pick emoji buttons: 👍 🔥 🎉 ❤️
- Posts show username and timestamp
- Recent reactions served from **Redis cache** for speed
- Data source indicator: `Last load: 2 ms · Source: MySQL`

---

## 📁 Repository Structure

```
3-tier-app/
├── frontend/              # Frontend application (HTML/CSS/JS)
├── backend/               # Backend API server (Node.js/Python)
├── helm/                  # Helm chart for Kubernetes deployment
├── k8s/                   # Raw Kubernetes manifests
│   ├── deployments/
│   ├── services/
│   └── ingress/
├── docs/                  # Documentation & screenshots
├── .gitignore
├── azure-pipelines.yml    # Azure DevOps CI/CD pipeline definition
├── docker-compose.yml     # Local development compose file
├── docker-compose.prod.yml# Production compose configuration
├── sc.yaml                # StorageClass manifest
├── tls.crt                # TLS certificate (for Ingress)
├── tls.key                # TLS private key
└── README.md
```

---

## 🔄 CI/CD Pipeline

The pipeline is defined in **`azure-pipelines.yml`** and runs on Azure DevOps.

### Pipeline Stages

```
┌─────────────────────┐      ┌──────────────────────┐
│  Build and push to  │─────►│  Deploy to EC2        │
│  Docker Registry    │      │  (Skipped if not EC2) │
│  ✅ 2 jobs  2m 12s │      └──────────────────────┘
│  📦 1 artifact      │
└─────────────────────┘      ┌──────────────────────┐
                             │  Deploy to Kubernetes │
                             │  (Helm-based deploy)  │
                             │  ✅ 1 job  25-26s     │
                             └──────────────────────┘
                             ┌──────────────────────┐
                             │  Deploy to Kubernetes │
                             │  (GitOps/ArgoCD sync) │
                             │  Skipped if conditions│
                             └──────────────────────┘
```

### Stage Details

**Stage 1 — Build and push to Docker Registry**
- Builds Docker images for frontend and backend
- Tags and pushes to Docker registry
- Publishes 1 artifact
- Runs on every commit to `main` branch

**Stage 2 — Deploy to EC2 (Docker)**
- Deploys using Docker Compose on an EC2 instance
- Skipped when conditions don't match (e.g., Kubernetes path selected)

**Stage 3 — Deploy to Kubernetes**
- Deploys using Helm chart to the Kubernetes cluster
- Updates image tags in Helm values
- Triggers ArgoCD sync via GitOps

**Stage 4 — Deploy to Kubernetes (alternate)**
- Conditional stage for alternative Kubernetes deployment targets

### Pipeline Trigger
- Branch: `main`
- Manually triggerable from Azure DevOps UI
- Author: Mina John Emil Fouad

---

## ☸️ Kubernetes Deployment

The application runs as a multi-service deployment on Kubernetes:

### Services Deployed

| Service | Type | Description |
|---------|------|-------------|
| `frontend-svc` | ClusterIP | Serves the frontend application |
| `mood-svc` | ClusterIP | Handles mood voting API |
| `reactions-svc` | ClusterIP | Handles reaction wall API |
| `mysql-svc` | ClusterIP | MySQL database service |
| `redis-svc` | ClusterIP | Redis cache service |
| `buzzboard-ingress` | Ingress | Routes external traffic |
| `buzzboard-ingress-api` | Ingress | Routes API traffic |

### Pods

| Pod | Status | Replicas |
|-----|--------|---------|
| `frontend-bb4cdd868-4gtqk` | ✅ Running | 1/1 |
| `mood-6f57fcc8c7-qh2tl` | ✅ Running | 1/1 |
| `reactions-65974989...` | ✅ Running | 1/1 |
| `mysql-0` | ✅ Running | 1/1 |
| `redis-0` | ✅ Running | 1/1 |

### Ingress
- **Host:** `https://buzzboard.local`
- **TLS:** Enabled with self-signed certificate
- **Load Balancer IP:** `1/2.18.0.2`

### Helm Chart

Deployment is managed via a Helm chart located in `helm/`:

```bash
# Install/upgrade
helm upgrade --install buzzboard ./helm \
  --set image.tag=<TAG> \
  --namespace default

# Check status
helm status buzzboard
```

---

## 🔁 GitOps with ArgoCD

The project uses **ArgoCD v3.3.6** for GitOps continuous delivery.

### Application: `3tier-app`

| Property | Value |
|----------|-------|
| **App Health** | ✅ Healthy |
| **Sync Status** | ✅ Synced to `main` (d125f3b) |
| **Auto Sync** | Enabled |
| **Resources** | 32 Synced, 0 OutOfSync |

### How GitOps Works

```
Developer pushes code
        │
        ▼
Azure DevOps Pipeline runs
  → Builds Docker image
  → Pushes to registry
  → Updates Helm values (image tag)
  → Commits to Git repo
        │
        ▼
ArgoCD detects Git change
  → Auto-syncs Kubernetes cluster
  → Applies Helm chart changes
  → All 32 resources synced ✅
```

### ArgoCD Resource Tree

```
Cloud (LoadBalancer)
└── 1/2.18.0.2 (Service)
    ├── buzzboard-ingress-api (Ingress)
    │   ├── mood-svc ──► mood pod (running 1/1)
    │   ├── reactions-svc ──► reactions pod (running 1/1)
    │   └── frontend-svc ──► frontend pod (running 1/1)
    └── buzzboard-ingress (Ingress)
        └── frontend-svc
mysql-svc ──► mysql-0 (StatefulSet pod)
redis-svc ──► redis-0 (StatefulSet pod)
```

---

## 🚀 Getting Started

### Prerequisites

- Docker & Docker Compose
- Kubernetes cluster (minikube / kind / cloud)
- `kubectl` configured
- Helm 3.x
- ArgoCD installed on cluster (optional, for GitOps)

### Local Development

```bash
# Clone the repo
git clone https://github.com/mina-john-emil/3-tier-app.git
cd 3-tier-app

# Start all services locally
docker-compose up -d

# Access the app
open http://localhost:3000
```

### Production (Kubernetes + Helm)

```bash
# Add TLS secret
kubectl create secret tls buzzboard-tls \
  --cert=tls.crt \
  --key=tls.key

# Deploy with Helm
helm upgrade --install buzzboard ./helm \
  --namespace default \
  --set mysql.enabled=true \
  --set redis.enabled=true

# Add to /etc/hosts (local cluster)
echo "127.0.0.1 buzzboard.local" >> /etc/hosts

# Access
open https://buzzboard.local
```

### Using ArgoCD (GitOps)

```bash
# Apply ArgoCD application manifest
kubectl apply -f k8s/argocd-app.yaml

# ArgoCD will auto-sync from main branch
# Monitor at: https://<argocd-server>/applications/3tier-app
```

---

## 📊 App Demo Flow

1. Navigate to `https://buzzboard.local`
2. **Sign up** with username `mina` and a password
3. **Sign in** to your account
4. Go to **Mood of the Day** → vote on how the room feels
5. Go to **Reaction Wall** → post a message or emoji reaction
6. See live vote counts and reaction history



## 📄 License

This project is created for educational/tutorial purposes.


