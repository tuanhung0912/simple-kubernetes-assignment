# ☸️ Simple Kubernetes Assignment

> A hands-on project demonstrating how to deploy a web application on Kubernetes using **Pod**, **Deployment**, and **NodePort Service** configurations.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Project Structure](#-project-structure)
- [Configuration Details](#-configuration-details)
  - [Client Pod](#1-client-pod-client-podyaml)
  - [Client Deployment](#2-client-deployment-client-deploymentyaml)
  - [Client NodePort Service](#3-client-nodeport-service-client-node-portyaml)
- [Prerequisites](#-prerequisites)
- [Deployment Guide](#-deployment-guide)
- [Access & Verification](#-access--verification)
- [Useful Kubernetes Commands](#-useful-kubernetes-commands)
- [Core Concepts](#-core-concepts)
- [Author](#-author)

---

## 🎯 Overview

This project illustrates how to deploy a web client application on **Kubernetes** using two fundamental resource types:

| Component | Description |
|---|---|
| **Pod** | The smallest deployable unit in K8s, running the `multi-client` container |
| **Deployment** | A higher-level controller that manages Pod replicas, rolling updates, and self-healing |
| **NodePort Service** | Exposes the application outside the cluster through a specific port on each Node |

The application uses the `stephengrider/multi-client` Docker image — a React client application running on port `3000`.

---

## 📁 Project Structure

```
simplek8s/
├── client-pod.yaml          # Pod configuration for the client application
├── client-deployment.yaml   # Deployment configuration (manages Pods with rolling updates)
├── client-node-port.yaml    # NodePort Service configuration to expose the app
└── README.md                # Project documentation (this file)
```

---

## 🔧 Configuration Details

### 1. Client Pod (`client-pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers:
    - name: client
      image: stephengrider/multi-client
      ports:
        - containerPort: 3000
```

| Field | Value | Description |
|---|---|---|
| `apiVersion` | `v1` | Stable API version for the Pod resource |
| `kind` | `Pod` | Resource type — the smallest deployable unit in K8s |
| `metadata.name` | `client-pod` | Unique identifier for the Pod within the namespace |
| `metadata.labels.component` | `web` | Label used by the Service selector to discover and connect to this Pod |
| `spec.containers[0].name` | `client` | Name of the container inside the Pod |
| `spec.containers[0].image` | `stephengrider/multi-client` | Docker image pulled from Docker Hub |
| `spec.containers[0].ports[0].containerPort` | `3000` | The port the React application listens on inside the container |

---

### 2. Client Deployment (`client-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: client
          image: stephengrider/multi-client
          ports:
            - containerPort: 3000
```

| Field | Value | Description |
|---|---|---|
| `apiVersion` | `apps/v1` | API group for Deployment resources |
| `kind` | `Deployment` | Resource type — manages ReplicaSets and Pods |
| `metadata.name` | `client-deployment` | Unique identifier for the Deployment |
| `spec.replicas` | `1` | Number of Pod replicas to maintain |
| `spec.selector.matchLabels` | `component: web` | Selector to identify which Pods this Deployment manages |
| `spec.template.metadata.labels` | `component: web` | Labels applied to Pods created by this Deployment |
| `spec.template.spec.containers[0].name` | `client` | Name of the container inside the Pod |
| `spec.template.spec.containers[0].image` | `stephengrider/multi-client` | Docker image pulled from Docker Hub |
| `spec.template.spec.containers[0].ports[0].containerPort` | `3000` | The port the React application listens on |

#### 🔄 Deployment vs Pod — Key Differences:

| Feature | Pod (`client-pod.yaml`) | Deployment (`client-deployment.yaml`) |
|---|---|---|
| **Self-healing** | ❌ If deleted, it's gone | ✅ Automatically recreates Pods |
| **Rolling updates** | ❌ Must delete and recreate manually | ✅ Updates with zero downtime |
| **Scaling** | ❌ Single instance only | ✅ Scale with `replicas` field |
| **Rollback** | ❌ No version history | ✅ Rollback to previous versions |
| **Production use** | ⚠️ Learning/testing only | ✅ Recommended for production |

---

### 3. Client NodePort Service (`client-node-port.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: client-node-port
spec:
  type: NodePort
  ports:
    - port: 3050
      targetPort: 3000
      nodePort: 31515
  selector:
    component: web
```

| Field | Value | Description |
|---|---|---|
| `apiVersion` | `v1` | Stable API version for the Service resource |
| `kind` | `Service` | Resource type — manages networking for Pods |
| `metadata.name` | `client-node-port` | Unique identifier for the Service |
| `spec.type` | `NodePort` | Service type that allows external access to the cluster |
| `spec.ports[0].port` | `3050` | Internal Service port (ClusterIP port) |
| `spec.ports[0].targetPort` | `3000` | Port on the Pod that the Service forwards traffic to |
| `spec.ports[0].nodePort` | `31515` | Port exposed on the Node for external access (valid range: 30000–32767) |
| `spec.selector.component` | `web` | Matches the label `component: web` on the Pod |

#### 🔑 Understanding the 3 Port Types:

```
External Traffic → nodePort (31515) → port (3050) → targetPort (3000) → Container
```

| Port | Purpose |
|---|---|
| **nodePort** (`31515`) | Externally accessible port — users connect through this port |
| **port** (`3050`) | Internal Service port within the cluster |
| **targetPort** (`3000`) | Actual application port inside the container |

---

## 💻 Prerequisites

- **Docker Desktop** (with built-in Kubernetes) or **Minikube**
- **kubectl** — Kubernetes command-line tool
- Internet connection (to pull the Docker image on first run)

### Verify installation:

```bash
# Check kubectl version
kubectl version --client

# Check cluster status
kubectl cluster-info

# Check available nodes
kubectl get nodes
```

---

## 🚀 Deployment Guide

### Step 1: Clone the repository

```bash
git clone https://github.com/Tuanhung0912/Simple-Kubernetes-Assignment.git
cd Simple-Kubernetes-Assignment
```

### Step 2: Ensure the Kubernetes cluster is running

If using **Docker Desktop**: Go to Settings → Kubernetes → ✅ Enable Kubernetes

If using **Minikube**:

```bash
minikube start
```

### Step 3: Apply Kubernetes configurations

**Option A** — Using Deployment (recommended):

```bash
# Deploy the Deployment (manages Pods automatically)
kubectl apply -f client-deployment.yaml

# Deploy the Service
kubectl apply -f client-node-port.yaml
```

**Option B** — Using standalone Pod (for learning):

```bash
# Deploy the Pod directly
kubectl apply -f client-pod.yaml

# Deploy the Service
kubectl apply -f client-node-port.yaml
```

Or apply all configurations at once:

```bash
kubectl apply -f .
```

### Step 4: Verify deployment status

```bash
# Check Deployment status
kubectl get deployments

# Check Pod status
kubectl get pods

# Check Service status
kubectl get services
```

Wait until the Pod status shows **Running**:

```
NAME                                 READY   STATUS    RESTARTS   AGE
client-deployment-7d4b8f6c9f-x2k5p   1/1     Running   0          30s
```

---

## 🌐 Access & Verification

### Access the application:

| Environment | URL |
|---|---|
| **Docker Desktop** | [http://localhost:31515](http://localhost:31515) |
| **Minikube** | `http://<minikube-ip>:31515` |

To get the Minikube IP:

```bash
minikube ip
```

Or open the service directly:

```bash
minikube service client-node-port
```

---

## 📝 Useful Kubernetes Commands

### Deployment Management

```bash
# List all Deployments
kubectl get deployments

# View detailed Deployment information
kubectl describe deployment client-deployment

# Scale the Deployment (e.g., to 3 replicas)
kubectl scale deployment client-deployment --replicas=3

# Update the image (triggers a rolling update)
kubectl set image deployment/client-deployment client=stephengrider/multi-client:v2

# Check rollout status
kubectl rollout status deployment/client-deployment

# Rollback to the previous version
kubectl rollout undo deployment/client-deployment

# View rollout history
kubectl rollout history deployment/client-deployment

# Delete the Deployment
kubectl delete deployment client-deployment
```

### Pod Management

```bash
# List all Pods
kubectl get pods

# View detailed Pod information
kubectl describe pod client-pod

# View Pod logs
kubectl logs client-pod

# Access shell inside the Pod
kubectl exec -it client-pod -- sh

# Delete the Pod
kubectl delete pod client-pod
```

### Service Management

```bash
# List all Services
kubectl get services

# View detailed Service information
kubectl describe service client-node-port

# Delete the Service
kubectl delete service client-node-port
```

### General Operations

```bash
# View all resources
kubectl get all

# Apply all config files in the directory
kubectl apply -f .

# Delete all resources defined in config files
kubectl delete -f .

# View cluster events sorted by timestamp
kubectl get events --sort-by='.lastTimestamp'
```

---

## 📚 Core Concepts

### What is a Pod?

- The **smallest deployable unit** in Kubernetes
- Contains **one or more containers** that share the same network and storage
- Each Pod gets its **own IP address** within the cluster
- Pods are **ephemeral** — if deleted, they will not automatically recreate themselves

### What is a Service?

- Provides **stable networking** for a group of Pods
- Uses **selectors** to find Pods based on their labels
- Available types: **ClusterIP**, **NodePort**, **LoadBalancer**, **ExternalName**

### Service Types Comparison:

| Type | Description | Accessibility |
|---|---|---|
| **ClusterIP** | Default type, accessible only within the cluster | Internal only |
| **NodePort** | Opens a port on each Node (range: 30000–32767) | External access |
| **LoadBalancer** | Provisions a load balancer from the cloud provider | Public / Internet |
| **ExternalName** | Maps the service to an external DNS name | DNS redirect |

### What is a Deployment?

- A **higher-level controller** that manages Pods through ReplicaSets
- Ensures the **desired number of replicas** are always running
- Supports **rolling updates** — update containers with zero downtime
- Provides **rollback** capability to revert to a previous version
- Pods created by a Deployment are **automatically recreated** if they crash or get deleted

### Why Do We Need Services?

- Pod IPs are **not static** — they change every time a Pod is recreated
- Services provide a **stable DNS name and IP** to reliably access Pods
- Services automatically **load balance** traffic across all Pods matching the selector

---

## 👤 Author

**Tuanhung0912** — [GitHub Profile](https://github.com/Tuanhung0912)

---

<p align="center">
  <i>📘 This project was created as part of the Docker & Kubernetes learning journey</i>
</p>
