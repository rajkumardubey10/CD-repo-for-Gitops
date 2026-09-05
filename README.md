# 📦 CD Repository – GitOps-Based Kubernetes Delivery with Argo CD

## Repository Purpose

This repository represents the **Continuous Delivery (CD) layer** of the system and follows **GitOps principles** for Kubernetes deployments.

It contains **Kubernetes manifests** that define the desired state of the application. **Argo CD continuously reconciles this repository with the Kubernetes cluster**, ensuring that the running state matches what is declared in Git.

> This repository does **not** contain CI logic.  
> The CI pipeline updates the Kubernetes manifests in this repository, and Argo CD applies those changes to the cluster.

---

## 🔁 GitOps Deployment Model

The deployment process follows a **pull-based GitOps model**.

- CI pipeline builds and validates the application
- CI pipeline builds and pushes the Docker image
- CI pipeline updates the image tag in this repository
- Argo CD detects the Git change automatically
- Argo CD synchronizes the desired state with Kubernetes
- Kubernetes reconciles the application workloads
- Staging deployment is verified before production promotion
- Production deployment requires manual approval
- Slack provides CI/CD execution notifications

Git acts as the **single source of truth** for Kubernetes deployments.

### Complete Deployment Flow

```text
Developer
    ↓
CI Repository
    ↓
PR Validation
    ↓
PR Review & Merge
    ↓
Merge CI Pipeline
    ↓
SonarQube Quality Gate
    ↓
Docker Build & Push
    ↓
Trivy Image Scan
    ↓
Update Image Tag in CD Repository
    ↓
Argo CD Detects Change
    ↓
Staging Deployment
    ↓
Rollout Verification
    ↓
HTTP Smoke Test
    ↓
Manual Production Approval
    ↓
Production Deployment
    ↓
Slack Notification
```

---

## 🚀 Argo CD Application Overview

The Argo CD Applications view shows the Kubernetes applications managed through this GitOps repository.

<img width="1366" height="768" alt="Argocd_application_screnshot" src="https://github.com/user-attachments/assets/4c305a36-b21e-4746-be30-28087161861d" />

### Staging

- **Application:** `ocatabyteproj`
- **Target Revision:** `assessment`
- **Path:** `K8`
- **Namespace:** `default`
- **Status:** **Healthy**
- **Sync Status:** **Synced**

### Production

- **Application:** `ocatabyteprojproduction`
- **Target Revision:** `assessment`
- **Path:** `K8/production`
- **Namespace:** `production`
- **Status:** **Healthy**
- **Sync Status:** **Synced**

This demonstrates that Argo CD is managing separate **staging and production environments** from the GitOps repository.

---

## 🌳 Argo CD Resource Graph

The Argo CD resource graph provides visibility into how the production application is deployed and related Kubernetes resources.

<img width="1366" height="768" alt="Argocd_graph_screeshot" src="https://github.com/user-attachments/assets/c1caade1-0150-4404-b54f-724e2ef2a7e1" />

The production resource hierarchy includes:

```text
Argo CD Application
        ↓
Production Namespace
        ↓
Service
        ↓
Deployment
        ↓
ReplicaSet
        ↓
Pods
```

This makes it possible to verify the complete Kubernetes resource chain from the Argo CD application down to the running Pods.

---

## 🧪 Staging Deployment Verification

After Argo CD synchronizes the updated manifest, the staging deployment is verified before production promotion.

The verification process checks:

```bash
kubectl rollout status deployment/fastapi-deployment --timeout=180s
```

The deployed image can also be verified using:

```bash
kubectl get deployment fastapi-deployment -o=jsonpath='{.spec.template.spec.containers[0].image}'
```

The application health endpoint is then tested using an HTTP smoke test:

```bash
curl -fsS http://localhost:30000/health
```

Expected response:

```json
{"status":"ok"}
```

This provides an additional validation layer before production deployment.

---

## 🔐 Manual Production Approval

Production deployment is protected using a **GitHub Environment approval gate**.

<img width="1366" height="768" alt="manual_approval" src="https://github.com/user-attachments/assets/7e7fabc2-2d9e-47ab-ad18-08b34898c0bf" />

The production deployment does not proceed automatically until an authorized reviewer approves the deployment.

This provides a controlled promotion path:

```text
Staging Deployment
       ↓
Rollout Verification
       ↓
HTTP Smoke Test
       ↓
Manual Approval
       ↓
Production Deployment
```

If the deployment is rejected, the production deployment job does not execute.

---

## 🏭 Production Environment

Production Kubernetes resources are maintained separately from staging.

The production manifests are located under:

```text
K8/
└── production/
    ├── deployment.yml
    └── service.yml
```

The production deployment uses:

- Dedicated `production` namespace
- Separate Kubernetes Deployment
- Separate Kubernetes Service
- Multiple replicas
- Rolling update strategy
- Readiness probes
- Liveness probes
- Docker registry authentication through `imagePullSecrets`
- Pod anti-affinity for workload distribution

This separation prevents staging configuration from directly becoming the production configuration.

---

## 🔄 CI and CD Repository Relationship

The CI and CD repositories have clearly separated responsibilities.

| Repository | Responsibility |
|---|---|
| **CI Repository** | Code validation, testing, security scanning, Docker build, image push and GitOps manifest update |
| **CD Repository** | Kubernetes manifests and desired deployment state |
| **Argo CD** | Continuously reconciles Git state with Kubernetes |
| **Kubernetes** | Runs and manages the application workloads |

The overall relationship is:

```text
CI Repository
     │
     │ Updates image tag
     ↓
CD Repository
     │
     │ Git change detected
     ↓
Argo CD
     │
     │ Reconciliation
     ↓
Kubernetes Cluster
```

The detailed CI/CD pipeline implementation is documented in the CI repository README.

---

## 🛡️ Deployment Control and Rollback

The deployment process is designed to provide controlled and traceable releases.

### Deployment Controls

- Git-based desired state
- Argo CD automated synchronization
- Separate staging and production environments
- Staging rollout verification
- HTTP smoke testing
- Manual production approval
- Kubernetes rolling updates
- Readiness and liveness probes

### Rollback

Because deployment state is stored in Git, previous application versions can be restored by reverting the corresponding manifest change.

Argo CD then reconciles the reverted Git state back to Kubernetes.

```text
Git Commit
    ↓
Argo CD Sync
    ↓
Kubernetes Deployment
    ↓
Application Version
```

Rollback:

```text
Revert Git Commit
    ↓
Argo CD Detects Change
    ↓
Argo CD Sync
    ↓
Previous Application Version
```

---

## 📁 Repository Structure

```text
.
├── K8/
│   ├── deployment.yml
│   ├── service.yml
│   └── production/
│       ├── deployment.yml
│       └── service.yml
├── Screenshots/
│   ├── argocd-applications.png
│   ├── argocd-production-resource-graph.png
│   └── approving_deployment.png
├── kind-config.yaml
└── README.md
```

---

## 🎯 Key Takeaway

This repository demonstrates a **GitOps-based Continuous Delivery model** where:

- Kubernetes desired state is stored in Git
- CI updates the deployment image reference
- Argo CD continuously reconciles Git with Kubernetes
- Staging and production are managed separately
- Staging is verified before production promotion
- Production requires explicit manual approval
- Kubernetes provides rolling deployment and health checks
- Git provides deployment traceability and rollback capability

The CI repository contains the detailed **CI/CD pipeline implementation**, while this repository focuses specifically on **Kubernetes manifests, GitOps delivery, and Argo CD-based deployment management**.
