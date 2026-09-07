Kubernetes Microservices Deployment & Observability Stack

This repository contains my local Kubernetes deployment for a Guestbook-style microservice architecture (NGINX Frontend + Redis Stateful Backend) built on Minikube.

The goal of this project was to implement production-grade K8s patterns: storage persistence, horizontal autoscaling, custom domain ingress, and full cluster monitoring using Prometheus and Grafana.
# 🏗️ System Architecture

    Frontend: NGINX serving static files, exposed internally via ClusterIP and externally via Ingress.

    Backend: Redis with password authentication (Secret) and configuration parameters (ConfigMap).

    Storage: PersistentVolumeClaim (PVC) attached to Redis to persist data across pod restarts.

    Autoscaling: HorizontalPodAutoscaler (HPA) targeting CPU utilization.

    Monitoring: Prometheus Operator & Grafana deployed via Helm (kube-prometheus-stack).

# 📁Repository Layout
manifests/
├── 01-config.yaml       # ConfigMaps & Secrets (Redis credentials)
├── 02-pvc.yaml          # PersistentVolumeClaim (1Gi RWO)
├── 03-redis.yaml        # Redis Deployment & ClusterIP Service
├── 04-production.yaml   # Frontend Deployment (Limits, Probes) & HPA
└── 05-ingress.yaml      # NGINX Ingress routing for myapp.local

# 🚀How to Run Locally
## 1. Prerequisites:

Ensure you have minikube, kubectl, and helm installed.

minikube start --driver=docker
minikube addons enable ingress
minikube addons enable metrics-server

## 2. Deploy Manifests:

kubectl apply -f manifests/01-config.yaml
kubectl apply -f manifests/02-pvc.yaml
kubectl apply -f manifests/03-redis.yaml
kubectl apply -f manifests/04-production.yaml
kubectl apply -f manifests/05-ingress.yaml

## 3. Setup Ingress Domain

Add Minikube's IP to your /etc/hosts:

echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

Access the application at [http://myapp.local](http://myapp.local).


# 📊 Monitoring Setup (Prometheus & Grafana)

I installed the Prometheus stack using Helm to monitor cluster resources in real time:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

## To access Grafana:

kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

User: admin

Password: Get via kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d

# 🧪 Validation & Tests
Data Persistence Test

Verified that data stored in Redis survives pod deletions:

## 1. Set key in Redis
kubectl exec -it deployment/redis-backend -- redis-cli -a MySuperSecretPassword123 SET user:1 "Aymane"
kubectl exec -it deployment/redis-backend -- redis-cli -a MySuperSecretPassword123 SAVE

## 2. Kill the Redis pod
kubectl delete pod -l app=redis

## 3. Read key from new pod
kubectl exec -it deployment/redis-backend -- redis-cli -a MySuperSecretPassword123 GET user:1
## Result: "Aymane"


👨‍💻 Author

Aymane BOUJDI

DevOps / Cloud Native Enthusiast
