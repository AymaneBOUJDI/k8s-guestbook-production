# ☸️ Production-Ready Microservices Architecture on Kubernetes

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh/)
[![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)
[![NGINX](https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://www.nginx.com/)

An end-to-end, production-grade microservices application deployed on a local **Kubernetes (Minikube)** cluster. This project demonstrates core Cloud-Native practices including high availability, dynamic autoscaling (HPA), persistent storage, ingress routing, secret management, and full observability using **Prometheus** and **Grafana**.

---

## 📐 Architecture Overview

```text
                                [ User Browser ]
                                       │
                                       ▼
                            [ NGINX Ingress Controller ]
                               (Domain: myapp.local)
                                       │
                 ┌─────────────────────┴─────────────────────┐
                 │                                           │
                 ▼                                           ▼
      [ Web Frontend Deploy ]                     [ Redis Backend Deploy ]
    (Replicas: 2-10 via HPA)                      (Replicas: 1 + Auth)
                 │                                           │
       (Liveness & Readiness)                       (Persistent Volume Claim)
                 │                                           │
                 ▼                                           ▼
       [ Frontend Pods ] ──(ClusterIP)──> [ Redis Pods ] ──> [ Persistent Volume (1Gi) ]
                 │                                           │
                 └─────────────────────┬─────────────────────┘
                                       │
                                       ▼
                       [ Prometheus & Grafana Stack ]
                           (Cluster Observability)

🔥 Key Technical Features
1. High Availability & Auto-Healing

    Deployment strategies with multiple replicas (replicas: 2).

    Automatic Pod recovery managed by Kubernetes Deployment controllers upon failure.

2. Dynamic Horizontal Pod Autoscaling (HPA)

    Configured target CPU utilization threshold (50%).

    Automatic scaling from 2 up to 10 replicas based on real-time resource consumption.

3. Data Persistence (Stateful Backend)

    Integrated PersistentVolumeClaim (PVC) using ReadWriteOnce access mode to attach local storage.

    Database state survival verified across Pod deletions and cluster restarts.

4. Decoupled Configurations & Security

    ConfigMaps: Injected environment variables (APP_ENV, REDIS_PORT) across workloads.

    Secrets: Encrypted base64 password management for Redis database authentication (REDIS_PASSWORD).

5. Ingress Networking & Custom Domain

    NGINX Ingress Controller configured for path-based HTTP routing.

    Local DNS binding (myapp.local) eliminating exposed arbitrary NodePorts.

6. Production Health Checks & Resource Management

    Configured Liveness and Readiness Probes to prevent bad traffic routing.

    Explicitly set Resource Requests & Limits (cpu: 50m-100m, memory: 64Mi-128Mi) to prevent OOMKills.

7. Full Observability (Monitoring & Alerting)

    Deployed kube-prometheus-stack via Helm.

    Real-time cluster metrics collection (CPU, Memory, Network) visualized through pre-configured Grafana Dashboards.

🛠️ Tech Stack & Tools

    Orchestration: Kubernetes, Minikube

    CLI Tools: kubectl, helm, minikube

    Containers: Docker, NGINX, Redis Alpine

    Networking & Ingress: NGINX Ingress Controller

    Monitoring & Observability: Prometheus Operator, Grafana

    OS Environment: Linux / Ubuntu

🚀 Quickstart Guide (Local Deployment)
Prerequisites

    Linux environment (Ubuntu / Debian recommended)

    Docker, Minikube, kubectl, and helm installed.

Step 1: Start the Kubernetes Cluster & Enable Addons
Bash

minikube start --driver=docker
minikube addons enable ingress
minikube addons enable metrics-server

Step 2: Clone the Repository
Bash

git clone [https://github.com/AymaneBOUJDI/k8s-guestbook-production.git](https://github.com/AymaneBOUJDI/k8s-guestbook-production.git)
cd k8s-guestbook-production

Step 3: Deploy Core Application Manifests
Bash

kubectl apply -f manifests/config.yaml
kubectl apply -f manifests/pvc.yaml
kubectl apply -f manifests/redis-v2.yaml
kubectl apply -f manifests/production.yaml
kubectl apply -f manifests/ingress.yaml

Step 4: Configure Local DNS

Bind Minikube's IP address to the custom domain:
Bash

echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

Visit http://myapp.local in your browser.
📊 Deploying Observability (Prometheus & Grafana)

    Add Helm Repository and Deploy Stack:

Bash

helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

    Access Grafana Dashboard:

Bash

# Port forward Grafana service
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

    Get Admin Password:

Bash

kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

    Open http://localhost:3000 in your browser. Navigating to Dashboards -> Kubernetes / Compute Resources / Namespace (Pods) shows real-time metrics for all application workloads.

🧪 Verification & Testing
Test 1: Verify Data Persistence
Bash

# Insert key into Redis
kubectl exec -it deployment/redis-backend -- redis-cli -a MySuperSecretPassword123 SET user:1 "Aymane"
kubectl exec -it deployment/redis-backend -- redis-cli -a MySuperSecretPassword123 SAVE

# Delete Pod to simulate crash
kubectl delete pod -l app=redis

# Fetch key from the newly created Pod
kubectl exec -it deployment/redis-backend -- redis-cli -a MySuperSecretPassword123 GET user:1
# Output: "Aymane"

Test 2: Verify HPA Auto-Scaling
Bash

# Watch HPA status in real time
kubectl get hpa -w

📁 Repository Structure
Plaintext

.
├── manifests/
│   ├── config.yaml          # ConfigMap and Secret definitions
│   ├── pvc.yaml             # PersistentVolumeClaim storage manifest
│   ├── redis-v2.yaml        # Redis Deployment & ClusterIP Service
│   ├── production.yaml     # Frontend Deployment (Requests/Limits/Probes) + HPA
│   └── ingress.yaml        # NGINX Ingress rules for myapp.local
└── README.md                # Project documentation

👨‍💻 Author

Developed by Aymane BOUJDI as a practical hands-on project to master Cloud-Native engineering and production Kubernetes orchestration.
