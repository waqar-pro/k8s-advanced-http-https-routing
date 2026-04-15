# 🌐 Kubernetes Ingress Lab
### Advanced HTTP/S Routing with NGINX Ingress Controller

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white)
![TLS](https://img.shields.io/badge/TLS-SSL-orange?style=for-the-badge&logo=letsencrypt&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-F7931E?style=for-the-badge&logo=kubernetes&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Architecture](#-architecture)
- [Lab Structure](#-lab-structure)
- [Prerequisites](#-prerequisites)
- [Tasks Completed](#-tasks-completed)
- [Routing Configuration](#-routing-configuration)
- [TLS Setup](#-tls-setup)
- [Testing Results](#-testing-results)
- [Key Learnings](#-key-learnings)
- [Commands Reference](#-commands-reference)
- [Troubleshooting](#-troubleshooting)
- [Cleanup](#-cleanup)

---

## 🎯 Overview

This lab demonstrates a **production-grade Kubernetes Ingress implementation** solving a real-world problem:

```
Problem:  Multiple apps, One IP — How to route traffic?
Solution: NGINX Ingress Controller with Path & Host based routing!

Before Ingress:               After Ingress:
App1 → IP:8081               myapps.local/app1  → App1
App2 → IP:8082               myapps.local/app2  → App2
App3 → IP:8083               api.myapps.local/  → App2
```

---

## 🏗️ Architecture

```
                        Internet
                           ↓
                    [myapps.local]
                           ↓
              ┌────────────────────────┐
              │  NGINX Ingress         │
              │  Controller            │
              │  (One Entry Point)     │
              └────────────────────────┘
                    ↓           ↓
            /app1 path      /app2 path
                ↓               ↓
        [App1 Service]    [App2 Service]
                ↓               ↓
        [App1 Pods x2]   [App2 Pods x2]
         (Blue Theme)    (Purple Theme)
```

---

## 📁 Lab Structure

```
kubernetes-ingress-lab/
│
├── 📄 app1-deployment.yaml       # App1 Deployment + Service + ConfigMap
├── 📄 app2-deployment.yaml       # App2 Deployment + Service + ConfigMap
│
├── 📄 ingress-basic.yaml         # Basic path-based routing (HTTP)
├── 📄 ingress-tls.yaml           # Ingress with TLS/HTTPS
├── 📄 ingress-advanced.yaml      # Advanced with subdomain routing
│
├── 📄 tls.key                    # Private Key (generated)
├── 📄 tls.crt                    # Self-signed Certificate (generated)
│
├── 📄 test-ingress.sh            # Automated testing script
└── 📄 README.md                  # This file
```

---

## ⚙️ Prerequisites

```bash
# Required tools
kubectl    ✅
minikube   ✅
openssl    ✅
curl       ✅

# Minikube addons
minikube addons enable ingress
```

---

## ✅ Tasks Completed

### Task 1 — Environment Setup
- [x] Verified Kubernetes cluster status
- [x] Enabled NGINX Ingress Controller via Minikube addon
- [x] Confirmed Ingress Controller running in `ingress-nginx` namespace

### Task 2 — Deploy Applications
- [x] Created `web-apps` namespace
- [x] Deployed App1 (Blue theme) with 2 replicas
- [x] Deployed App2 (Purple theme) with 2 replicas
- [x] Used ConfigMap to serve custom HTML (no image rebuild needed!)
- [x] Both apps exposed via ClusterIP Service

### Task 3 — Path-Based Routing
- [x] Created Ingress resource for `myapps.local`
- [x] Configured `/app1` → App1 Service
- [x] Configured `/app2` → App2 Service
- [x] Added `/etc/hosts` entry for local DNS resolution

### Task 4 — TLS/SSL Security
- [x] Generated self-signed certificate with OpenSSL
- [x] Stored certificate in Kubernetes TLS Secret
- [x] Updated Ingress to enforce HTTPS
- [x] Configured HTTP → HTTPS redirect (308)

### Task 5 — Advanced Routing
- [x] Added subdomain routing `api.myapps.local` → App2
- [x] Multi-host Ingress configuration
- [x] Resolved snippet annotation conflict error

### Task 6 — Testing & Verification
- [x] Created automated test script
- [x] Verified path-based routing
- [x] Verified HTTPS and redirect
- [x] Confirmed subdomain routing
- [x] Checked SSL certificate details

---

## 🔀 Routing Configuration

### Path-Based Routing
| URL Path | Routes To | App |
|----------|-----------|-----|
| `myapps.local/app1` | app1-service | App1 (Blue) |
| `myapps.local/app2` | app2-service | App2 (Purple) |
| `myapps.local/` | app1-service | App1 (Default) |

### Host-Based Routing
| Hostname | Routes To | App |
|----------|-----------|-----|
| `myapps.local` | app1-service | App1 |
| `api.myapps.local` | app2-service | App2 |

---

## 🔐 TLS Setup

```bash
# Step 1: Generate Private Key
openssl genrsa -out tls.key 2048

# Step 2: Create Certificate Request
openssl req -new -key tls.key -out tls.csr \
  -subj "/CN=myapps.local/O=myapps.local"

# Step 3: Self-Sign Certificate
openssl x509 -req -days 365 -in tls.csr \
  -signkey tls.key -out tls.crt

# Step 4: Store in Kubernetes Secret
kubectl create secret tls myapps-tls-secret \
  --cert=tls.crt \
  --key=tls.key \
  -n web-apps
```

### TLS Flow
```
Browser → HTTPS Request
            ↓
    Ingress (SSL Terminate)
    Certificate: myapps-tls-secret
            ↓
    HTTP → App1/App2 Pods
    (Internal traffic is HTTP)
```

---

## 🧪 Testing Results

### Automated Test Script Output
```bash
=== Ingress Testing Script ===

1. HTTP to HTTPS redirect:
HTTP Status: 308, Redirect: https://myapps.local/app1  ✅

2. HTTPS app1:
<title>Application 1</title>  ✅

3. HTTPS app2:
<title>Application 2</title>  ✅

4. API subdomain:
<title>Application 2</title>  ✅

5. SSL Certificate:
subject=CN=myapps.local  ✅

=== Testing Complete ===
```

---

## 💡 Key Learnings

### 1. Ingress vs Service Types
```
ClusterIP    → Only inside cluster
NodePort     → Specific port on node
LoadBalancer → Cloud load balancer (costs money)
Ingress      → Smart L7 router (path + host based) ✅
```

### 2. TLS Termination
```
Client → HTTPS → Ingress (decrypts here)
                     ↓
                 HTTP → Backend Pods
Apps don't need to handle SSL at all!
```

### 3. ConfigMap for HTML
```
No image rebuild needed!
Update ConfigMap → Pod picks up new HTML
Much more flexible than baking HTML into image
```

### 4. Real Errors Faced & Fixed
```
❌ configuration-snippet not allowed
   → Removed annotation, security feature of cluster

❌ path already defined conflict
   → Deleted old ingress before applying new one
```

---

## 📖 Commands Reference

```bash
# Enable ingress addon
minikube addons enable ingress

# Check ingress controller
kubectl get pods -n ingress-nginx

# Apply ingress
kubectl apply -f ingress-basic.yaml

# Check ingress
kubectl get ingress -n web-apps
kubectl describe ingress web-apps-ingress -n web-apps

# Get minikube IP
minikube ip

# Add to hosts file
echo "$(minikube ip) myapps.local" | sudo tee -a /etc/hosts

# Test routing
curl -k https://myapps.local/app1
curl -k https://myapps.local/app2
curl -k https://api.myapps.local/

# Check ingress controller logs
kubectl logs -n ingress-nginx \
  $(kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/component=controller \
  -o jsonpath='{.items[0].metadata.name}') --tail=50
```

---

## 🔧 Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `snippet annotation not allowed` | Security policy | Remove `configuration-snippet` annotation |
| `path already defined` conflict | Old ingress exists | `kubectl delete ingress --all -n web-apps` |
| Ingress not working | Controller not ready | `minikube addons enable ingress` |
| SSL warning in browser | Self-signed cert | Use `-k` flag in curl |
| 404 on path | Wrong pathType | Use `Prefix` not `Exact` |
| DNS not resolving | Hosts file missing | Add entry to `/etc/hosts` |

---

## 🧹 Cleanup

```bash
# Delete all ingress resources
kubectl delete ingress --all -n web-apps

# Delete applications
kubectl delete -f app1-deployment.yaml
kubectl delete -f app2-deployment.yaml

# Delete TLS secret
kubectl delete secret myapps-tls-secret -n web-apps

# Delete namespace
kubectl delete namespace web-apps

# Clean hosts file
sudo sed -i '/myapps.local/d' /etc/hosts

# Remove local files
rm -f tls.key tls.csr tls.crt *.yaml test-ingress.sh
```

---

## 👤 Author

**Your Name**
- 💼 LinkedIn: [your-linkedin](https://www.linkedin.com/feed/update/urn:li:activity:7449984850012463104/?originTrackingId=iVKYty5ZbSuypG6P94oY8Q%3D%3D)
- 🐙 GitHub: [your-github](https://github.com/waqar-pro?tab=repositories)

---

## 📜 License

This project is licensed under the MIT License.

---

> 💬 *"One entry point to rule them all."* — Every DevOps Engineer using Ingress 😄

⭐ **Helpful laga toh star zaroor karo!** ⭐
