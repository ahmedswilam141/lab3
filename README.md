# ☸️ Lab 3: Multi-Tenancy with Namespaces and Internal Routing

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Python](https://img.shields.io/badge/python-%2314354C.svg?style=for-the-badge&logo=python&logoColor=white)

---

## 📖 Scenario
Your team is adopting a "cluster per environment" strategy, but for cost reasons, you must use a single cluster for Development (`dev`) and Staging (`staging`) environments. You need to ensure logical separation and enable communication between two applications within the same namespace.

---

## 📝 Lab Description
Two namespaces (`dev` and `staging`) are created to simulate environment isolation. A two-tier application (frontend + backend) is deployed in each namespace. Services handle internal pod networking, and environment isolation is demonstrated by keeping each frontend communicating only with its own local `backend-service`.

---

## 🚀 Tasks Implemented

- [x] **Task 1:** Create two namespaces: `dev` and `staging`.
- [x] **Task 2:** Deploy a backend pod in `dev` and expose it via a ClusterIP service named `backend-service`.
- [x] **Task 3:** Deploy a frontend pod in `dev` configured to communicate with `backend-service`.
- [x] **Task 4:** Repeat the same deployment in `staging` with its own isolated `backend-service`.

---

## 📂 Project Structure

```text
.
├── dev-environment.yaml       # Backend + Frontend pods and backend service for dev
├── staging-environment.yaml   # Backend + Frontend pods and backend service for staging
└── README.md                  # Lab documentation
```

---

## 🛠️ Solution

### Build & Load Docker Images

```bash
# Build images
docker build -f Dockerfile.backend -t backend-app:latest .
docker build -f Dockerfile.frontend -t frontend-app:latest .

# Load into minikube
minikube image load backend-app:latest
minikube image load frontend-app:latest
```

---

### Task 1 — Create Namespaces

```bash
kubectl create namespace dev
kubectl create namespace staging
```

---

### Tasks 2 & 3 — Dev Environment

**dev-environment.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  namespace: dev
  labels:
    app: backend
spec:
  containers:
  - name: backend-container
    image: backend-app:latest
    imagePullPolicy: Never
    ports:
    - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: dev
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  namespace: dev
  labels:
    app: frontend
spec:
  containers:
  - name: frontend-container
    image: frontend-app:latest
    imagePullPolicy: Never
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f dev-environment.yaml
```

---

### Task 4 — Staging Environment

**staging-environment.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  namespace: staging
  labels:
    app: backend
spec:
  containers:
  - name: backend-container
    image: backend-app:latest
    imagePullPolicy: Never
    ports:
    - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: staging
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  namespace: staging
  labels:
    app: frontend
spec:
  containers:
  - name: frontend-container
    image: frontend-app:latest
    imagePullPolicy: Never
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f staging-environment.yaml
```

---

## ✅ Verification

### Pods and Services in `dev` and `staging`

```bash
kubectl get pods,svc -n dev
kubectl get pods,svc -n staging
```

![kubectl get pods,svc output][!k8s](./screenshots/get-pods-svc.png)

Both namespaces show:
- `pod/backend-pod` → Running
- `pod/frontend-pod` → Running
- `service/backend-service` → ClusterIP on port 8080

---

### Port Forward and Browser Test

```bash
kubectl port-forward pod/frontend-pod 8082:80 -n dev
```

Then open: **http://localhost:8082**

![Frontend Application in browser][!k8s](./screenshots/browser.png)

The frontend is accessible and shows buttons to test connectivity to the backend endpoints (`/`, `/health`, `/info`).

---

## 💡 Key Concepts Learned

| Concept | Description |
|---|---|
| **Namespaces** | Logical isolation of resources within a single cluster |
| **Multi-tenancy** | Running dev and staging on one cluster without interference |
| **ClusterIP Service** | Stable internal DNS name for pod-to-pod communication |
| **imagePullPolicy: Never** | Uses locally loaded images instead of pulling from a registry |
| **Service DNS** | Frontend reaches backend via `backend-service:8080` — Kubernetes resolves it within the namespace |
| **Environment Isolation** | Each namespace has its own `backend-service` — staging frontend never reaches dev backend |