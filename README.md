# kubernetes-mini-project


# 🚀 Kubernetes Production-Ready NGINX Application (Minikube)

This project demonstrates a production-style Kubernetes deployment of an NGINX-based web application using:

- Namespace isolation
- Deployment with 3 replicas
- ConfigMap (HTML content)
- ConfigMap (NGINX configuration)
- Secret (environment variable)
- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)
- Custom StorageClass
- Service (LoadBalancer)
- Minikube Tunnel
- Liveness & Readiness Probes
- Rolling Updates
- Self-Healing

This setup simulates a real-world production environment on Minikube.

---

# 🏗 Architecture


External Traffic
↓
LoadBalancer Service
↓
ClusterIP
↓
Deployment (3 replicas)
↓
Pods
├── ConfigMap (HTML)
├── ConfigMap (NGINX config)
├── Secret (Environment variable)
└── PVC → PV → StorageClass


---

# 🛠 Prerequisites

- Ubuntu / Linux
- Docker installed
- Minikube installed
- kubectl installed

---

# 🚀 Step 1 — Start Minikube

```bash
minikube start --driver=docker

Verify:

minikube status
📦 Step 2 — Create Namespace
kubectl create namespace web-app

Why?

Isolates resources

Simulates production multi-project environment

💾 Step 3 — Create StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: web-storage
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Retain
volumeBindingMode: Immediate

Apply:

kubectl apply -f storageclass.yaml

Why?

Defines how volumes are provisioned

Simulates cloud storage provisioning

💽 Step 4 — Create PersistentVolume (PV)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: web-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: web-storage
  hostPath:
    path: "/mnt/data"

Why?

Provides actual storage

Data persists even if pod restarts

📂 Step 5 — Create PersistentVolumeClaim (PVC)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-pvc
  namespace: web-app
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: web-storage
  resources:
    requests:
      storage: 500Mi

Why?

Pod does not request PV directly

PVC abstracts storage consumption

🗂 Step 6 — Create ConfigMap (HTML)
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
  namespace: web-app
data:
  index.html: |
    <html>
    <head>
      <meta charset="UTF-8">
      <title>Production App</title>
    </head>
    <body>
      <h1>🚀 Welcome to Production NGINX App</h1>
      <p>This is served using ConfigMap!</p>
    </body>
    </html>

Why?

Externalizes configuration

No need to rebuild Docker image

🔐 Step 7 — Create Secret
kubectl create secret generic web-secret \
  --from-literal=APP_PASSWORD=SuperSecret123 \
  -n web-app

Why?

Secure storage for sensitive data

Injected as environment variable

🌐 Step 8 — Create NGINX ConfigMap (UTF-8 Fix)
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: web-app
data:
  default.conf: |
    server {
        listen 80;
        server_name _;
        charset utf-8;

        location / {
            root   /usr/share/nginx/html;
            index  index.html;
        }
    }

Why?

Ensures proper charset header

Fixes emoji encoding issue

🚀 Step 9 — Create Deployment (3 Replicas)

Key features:

3 replicas

Resource limits

Liveness probe

Readiness probe

Mount ConfigMap + PVC

Apply:

kubectl apply -f deployment.yaml

Why?

High availability

Self-healing

Health monitoring

🌍 Step 10 — Create Service (LoadBalancer)
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: web-app
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80

Why?

Exposes application externally

Enables load balancing across pods

🔄 Step 11 — Start Minikube Tunnel
sudo -E minikube tunnel

Why?

Simulates cloud LoadBalancer

Assigns external IP

🧪 Step 12 — Test Application
kubectl get svc -n web-app
curl http://<EXTERNAL-IP>
🔥 Features Demonstrated

Rolling updates

Self-healing (delete pod → auto recreate)

Load balancing

Persistent storage

Config externalization

Secure secrets management

Health monitoring

📚 Production Concepts Covered

ClusterIP vs LoadBalancer

Readiness vs Liveness probes

ConfigMap vs Secret

PV vs PVC

Rolling deployment strategy

Kubernetes self-healing mechanism

Minikube LoadBalancer simulation

🏁 Conclusion

This project simulates a real-world production Kubernetes deployment with best practices applied.


---

# 🧠 NOW — EXPLANATION OF EVERYTHING (Deep Understanding)

Here’s what you actually built.

---

## 🔹 Why Namespace?

Production clusters host many apps.

Namespace provides:

- Isolation
- Resource separation
- RBAC boundary

---

## 🔹 Why StorageClass?

In cloud:

- AWS → EBS
- GCP → Persistent Disk

StorageClass tells Kubernetes:

> “How should storage be provisioned?”

Minikube uses hostPath.

---

## 🔹 Why PV + PVC?

Pods are ephemeral.

If pod dies → data inside container dies.

PVC ensures:

- Data persists beyond pod lifecycle.

---

## 🔹 Why ConfigMap?

Without ConfigMap:

- You must rebuild Docker image to change HTML.

With ConfigMap:

- You update config independently.

Separation of:
Code vs Configuration.

---

## 🔹 Why Secret?

Never hardcode passwords in YAML.

Secret:
- Base64 encoded
- Can be mounted or injected as env var.

---

## 🔹 Why Deployment?

Deployment provides:

- ReplicaSet management
- Rolling updates
- Scaling
- Self-healing

Without Deployment:
You only get one static pod.

---

## 🔹 Why 3 Replicas?

High availability.

If 1 pod crashes:
- 2 still serve traffic.
- Deployment auto-creates new one.

---

## 🔹 Why Liveness Probe?

Detects dead container.

If liveness fails:
- Pod restarted.

---

## 🔹 Why Readiness Probe?

Prevents traffic before pod is ready.

If readiness fails:
- Removed from Service endpoints.

---

## 🔹 Why Service?

Pods have dynamic IPs.

Service provides:

- Stable virtual IP
- Load balancing
- Service discovery

---

## 🔹 Why LoadBalancer?

Exposes service externally.

In cloud:
- Creates real LB.

In Minikube:
- Tunnel simulates it.

---

## 🔹 Why NGINX ConfigMap?

To fix:


Content-Type: text/html


We needed:


Content-Type: text/html; charset=utf-8


Server-level fix > HTML-level fix.

---

# 🏆 What You Have Achieved

You simulated:

- Production networking
- Production storage
- Production health checks
- Production deployment strategy
- External exposure
- Load balancing
- Self-healing
- Config management
- Secret management
