# Microservices Orchestrator (Kubernetes + Minikube)

This repository runs the whole microservices system inside a local Kubernetes cluster (Minikube):
- API Gateway
- Auth Service (+ Postgres)
- User Service (+ Postgres + Redis)
- Order Service (+ Postgres)
- Payment Service (+ MongoDB)
- Kafka + Zookeeper

## Prerequisites
Install:
- Docker
- kubectl
- Minikube

---

## 1) Start the cluster
```powershell
minikube start --driver=docker
minikube addons enable ingress
```
Check:
```powershell
kubectl get nodes -o wide
kubectl get pods -n ingress-nginx
```

---

## 2) Create Kubernetes Secrets from env files

### JWT secret
```powershell
kubectl create secret generic jwt-secret `
  --from-env-file=k8s/env/jwt.env `
  --dry-run=client -o yaml | kubectl apply -f -
```

### Auth DB secret
```powershell
kubectl create secret generic auth-db-secret `
  --from-env-file=k8s/env/uth-db.env `
  --dry-run=client -o yaml | kubectl apply -f -
```

### User DB secret
```powershell
kubectl create secret generic user-db-secret `
  --from-env-file=k8s/env/user-db.env `
  --dry-run=client -o yaml | kubectl apply -f -
```

### Order secret
```powershell
kubectl create secret generic order-secret `
  --from-env-file=k8s/env/order.env `
  --dry-run=client -o yaml | kubectl apply -f -
```
### Payment Secrets
```powershell
kubectl create secret generic payment-secret `
  --from-env-file=k8s/env/payment.env `
  --dry-run=client -o yaml | kubectl apply -f -
```
---

## 3) Deploy infra (databases + kafka) and services
Apply infra first, then services:
```powershell
kubectl apply -f k8s/infra
kubectl apply -f k8s/services
```

Wait until everything is Running:
```powershell
kubectl get pods -w
```

---

## 4) Ingress: access from your PC
Create ingress (example):
- host: microservices.innowise.local
- backend: api-gateway:8084

Check:
```powershell
kubectl get ingress
kubectl describe ingress api-gateway-ingress
```

### Add hosts entry (Windows)
Open Notepad as Administrator and edit:
C:\Windows\System32\drivers\etc\hosts

Add:
```
<INGRESS_NODE_IP> microservices.innowise.local
```

How to get the IP:
- use the Address field from: kubectl describe ingress api-gateway-ingress
- or: kubectl get nodes -o wide

Test:
```powershell
curl.exe -i http://microservices.innowise.local/actuator/health
```

Note:
- 404 on / is OK if your gateway has no route for root.
- Use real paths like /auth/login, /api/users, /api/orders, /payments, etc.

---

## 5) How to update services after CI/CD push (GHCR)
Your Deployments should use a stable tag like :develop and:
- imagePullPolicy: Always

After a new image is published to GHCR:
```powershell
kubectl rollout restart deploy/user-service
kubectl rollout restart deploy/order-service
kubectl rollout restart deploy/payment-service
kubectl rollout restart deploy/auth-service
kubectl rollout restart deploy/api-gateway
```

Verify image actually used:
```powershell
kubectl get deploy user-service -o jsonpath="{.spec.template.spec.containers[0].image}"
kubectl get pod -l app=user-service -o jsonpath="{.items[0].status.containerStatuses[0].image}"
kubectl get pod -l app=user-service -o jsonpath="{.items[0].status.containerStatuses[0].imageID}"
```

---

## 6) Stop / Delete (how to shut everything down)

### Option A: stop Minikube (fast, keeps cluster state)
```powershell
minikube stop
```

### Option B: delete all Kubernetes resources from this repo
```powershell
kubectl delete -f k8s/ingress --ignore-not-found
kubectl delete -f k8s/services --ignore-not-found
kubectl delete -f k8s/infra --ignore-not-found
```

### Option C: wipe the whole cluster (clean slate)
```powershell
minikube delete
```

---

## Payment env file (recommended)
Create: k8s/env/payment.env
Example:
- SPRING_DATA_MONGODB_URI=mongodb://payment-mongo:27017/payment_db
- APP_LIQUIBASE_URL=mongodb://payment-mongo:27017/payment_db
- SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
- EXTERNAL_RANDOM_URL=https://www.randomnumberapi.com/api/v1.0/random?min=1&max=100&count=1

Then create a secret/configmap from it and reference it in payment-service.yaml:

