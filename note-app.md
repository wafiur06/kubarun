```python
markdown_content = """# 🚀 Django Notes App - Kubernetes Deployment Guide

This repository contains the step-by-step guide and Kubernetes manifests to deploy a Django Notes application to a local Kubernetes cluster using **Kind (Kubernetes IN Docker)**.

## 📋 Prerequisites

Before you begin, ensure you have the following installed on your local machine:
- [Git](https://git-scm.com/)
- [Docker](https://www.docker.com/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Kind](https://kind.sigs.k8s.io/)

---

## 🛠️ Step-by-Step Deployment Process

### Step 1: Clone the Repository & Checkout Branch
First, clone the source code and switch to the development branch.

```

```text
File created successfully.

```bash
git clone [https://github.com/LondheShubham153/django-notes-app.git](https://github.com/LondheShubham153/django-notes-app.git)
cd django-notes-app
git checkout dev

```

### Step 2: Build and Tag the Docker Image

Build the Docker image using the provided `Dockerfile` and tag it with your username (e.g., `wafiur`).

```bash
docker build -t notes-app-k8s .
docker tag notes-app-k8s:latest wafiur/notes-app-k8s:latest

```

### Step 3: Load the Image into Kind Cluster

Since we are using a local `Kind` cluster, Kubernetes cannot pull the image from the internet. We need to load our local image directly into the cluster nodes.

```bash
kind load docker-image wafiur/notes-app-k8s:latest --name tws-cluster

```

*(Note: Replace `tws-cluster` with your actual Kind cluster name if it is different).*

### Step 4: Create Kubernetes Manifests

Create a directory named `k8s` inside your project and create the following YAML files.

**1. `namespace.yml**`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: notes-app

```

**2. `deployment.yml**`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  namespace: notes-app
  labels:
    app: notes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: wafiur/notes-app-k8s:latest
        imagePullPolicy: Never   # Prevents pulling from Docker Hub
        ports:
        - containerPort: 8000

```

**3. `service.yml**`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: notes-app-service
  namespace: notes-app
spec:
  selector:
    app: notes-app
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
  type: ClusterIP

```

### Step 5: Apply the Manifests

Apply the configuration files to your Kubernetes cluster in the following order:

```bash
cd k8s
kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml

```

### Step 6: Verify the Deployment

Check if the pods are running successfully.

```bash
kubectl get pods -n notes-app
kubectl get svc -n notes-app

```

*(If a pod is stuck in `ContainerCreating`, delete it using `kubectl delete pod <pod-name> -n notes-app` to force a restart with the loaded image).*

### Step 7: Access the Application (Port-Forwarding)

Forward the traffic from your local machine to the Kubernetes service to access the app in your browser.

```bash
kubectl port-forward service/notes-app-service -n notes-app 8002:8000 --address=0.0.0.0

```

*(Note: We used port `8002` locally to avoid conflicts if port `8000` is already in use).*

🌐 **Open your browser and visit:** [http://localhost:8002](https://www.google.com/search?q=http://localhost:8002)

---
