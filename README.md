# 📦 CD Repository – GitOps-Based Kubernetes Delivery with Argo CD

## Repository Purpose

This repository represents the **Continuous Delivery (CD) layer** of the system and follows
**GitOps principles** for Kubernetes deployments.

It contains **Kubernetes manifests** that define the desired state of the application.
**Argo CD continuously reconciles this repository with the Kubernetes cluster**, ensuring
that the running state always matches what is declared in Git.

> This repository does **not** contain CI logic.  
> CI pipelines update manifests here, and Argo CD applies those changes to the cluster.

---

## 🔁 GitOps Deployment Model

The deployment flow follows a **pull-based GitOps model**:

- CI pipeline builds and validates container images
- CI pipeline updates image tags in this repository
- Argo CD detects Git changes automatically
- Argo CD syncs the desired state to Kubernetes
- Kubernetes reconciles workloads based on Git state

Git remains the **single source of truth** for deployments.

---

## 🧭 Argo CD Application Overview

![Argo CD Application Tree View](./docs/images/argocd-application-tree.png)

This screenshot shows the **Argo CD Application Tree View**, providing a visual representation
of how Argo CD manages and tracks Kubernetes resources derived from this repository.

### What this shows:
- Application health status is **Healthy**
- Sync status is **Synced**
- Desired state is pulled from the `main` branch
- Auto-sync is enabled
- Kubernetes resources are continuously monitored

---

## 🌳 Argo CD Resource Tree & Health Status

![Argo CD Resource Graph](./docs/images/argocd-resource-graph.png)

This view shows the **resource hierarchy and live status** of the deployed application.

### What this proves:
- Kubernetes resources are correctly created from Git manifests
- Deployment, Service, ReplicaSet, and Pods are all healthy
- Replica count matches the desired state
- Argo CD continuously reconciles Git state with the cluster

This confirms **successful GitOps reconciliation**.

---

## 🚀 Application Running in Kubernetes

![Application Running from Kubernetes](./docs/images/application-running.png)

This screenshot shows the **FastAPI application running successfully in the Kubernetes cluster**,
served from a container image defined in this GitOps repository.

### What this proves:
- The Kubernetes Deployment is active and healthy
- The application is reachable through the Kubernetes Service
- The container image referenced in Git is running in the cluster
- End-to-end flow from CI → CD → Kubernetes is functioning correctly

---

## 🔐 Deployment Control & Safety

This GitOps setup enforces **strong deployment control and safety guarantees**:

- Argo CD operates in a **pull-based model**
- No direct `kubectl apply` access is required for deployments
- All deployment changes must go through Git
- Rollbacks are performed by reverting Git commits
- Cluster access is minimized, improving security and auditability

This ensures that deployments are:
- Controlled
- Auditable
- Repeatable
- Easy to roll back

---

## 🔗 Relationship to CI Repository

This CD repository is updated automatically by the **CI repository** after a successful
PR validation and merge pipeline execution.

### Responsibility separation:

| Repository | Responsibility |
|-----------|----------------|
| CI Repository | Build, scan, validate, and promote container images |
| CD Repository | Define the desired Kubernetes deployment state |
| Argo CD | Reconcile Git state to the Kubernetes cluster |

This clean separation follows **industry-standard GitOps architecture**.

---

## 📁 Repository Structure

```text
.
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── docs/
│   └── images/
│       ├── argocd-application-tree.png
│       ├── argocd-resource-graph.png
│       └── application-running.png
└── README.md

