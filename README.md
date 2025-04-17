# k8s-meatshop

## 📦 Introduction

This is a Helm chart for the project **elmanazala-meatshop**.

### 🔗 Source Repositories
- **Frontend:** [https://github.com/BRHM1/elmal7ma.git](https://github.com/BRHM1/elmal7ma.git)  
- **Backend:** [https://github.com/BRHM1/g-back.git](https://github.com/BRHM1/g-back.git)

### 📸 Docker Images
- **Frontend:** `borhom11/frontend:1.0.6`
- **Backend:** `borhom11/backend:1.0.1`

---

## Architecture diagram

![WhatsApp Image 2025-04-16 at 18 03 03_dfa75a72](https://github.com/user-attachments/assets/e02d4161-c2d5-44db-aa09-00592fee872d)

## ✅ Prerequisites

### 🧱 Storage Class
Local path provisioner:  
[https://github.com/rancher/local-path-provisioner](https://github.com/rancher/local-path-provisioner)

### 🌐 NGINX Ingress Controller

```bash
# Add the Helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install (with NodePort for bare metal/VMs)
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.hostNetwork=true  # Only for bare metal/VMs
```

### 📁 /etc/hosts Configuration

If you're running a kubeadm cluster, ensure that the IP of the node running the ingress controller pod is added to `/etc/hosts` with the following mappings:

```
<node-ip> frontend.com backend.com
```

This allows access to the applications via:
```
http://frontend.com:<ingress_controller_nodePort>
http://backend.com:<ingress_controller_nodePort>
```

#### 🪜 Steps to identify the node IP:

1. Execute: `kubectl get pods -n ingress-nginx -o wide`
2. SSH into the node running the ingress controller pod: `ssh <node-name>`
3. Run: `ip addr` and identify the IP of the correct interface
4. Edit `/etc/hosts` and add the entry as shown above

---

## ⚙️ Values.yaml Structure

### 🌐 Frontend

```yaml
frontend:
  deployment:
    name: <deployment-name>
    labels:
      <key>: <value>
    replicas: <num-of-pods>
    image: borhom11/frontend:1.0.5
  services:
    name: <frontend-service-name>
    port: <service-port>
  cm:
    name: <frontend-configmap-name>
```

### 🖥 Backend

```yaml
backend:
  deployment:
    name: <backend-deployment-name>
    labels:
      <key>: <value>
    replicas: <num-of-pods>
    image: eladwy/g-back
    secretName: <backend-secret-name>
  services:
    nodePort:
      name: <nodeport-service-name>
      port: <nodeport-port>
    clusterIP:
      name: <clusterip-service-name>
      port: <clusterip-port>
  secret:
    name: <backend-secret-name>
    db_name: (e.g., meatshop)
    port: (e.g., 3306)
    user: (e.g., root)
    password: <base64-encoded-password>
    host: <mysql-statefulset-pod>.<headless-service> (e.g., mysql-0.mysql)
    settings_module: bWVhdHNob3Auc2V0dGluZ3MuZGV2
    secret_key: Generate from https://djecrety.ir/ and paste the base64-encoded value here
    superuser_email: <superuser-email>
    superuser_username: <superuser-username>
    superuser_password: <superuser-password>
```

### 🛢 MySQL

```yaml
mysql:
  statefulset:
    name: <statefulset-name>
    labels:
      <key>: <value>
    image: mysql
    storageClass: <storage-class-name>
  secret:
    name: <mysql-secret-name>
    db: bWVhdHNob3A=
    root_password: <root-password>
    user: cm9vdA==
  cms:
    liveness:
      name: <liveness-configmap-name>
    readiness:
      name: <readiness-configmap-name>
  service:
    name: <mysql-headless-service-name>
  job:
    name: <mysql-init-job-name>
```

### 🌍 Ingress

```yaml
ingress:
  name: <ingress-name>
  frontend_host: <frontend-hostname-in-/etc/hosts>
  backend_host: <backend-hostname-in-/etc/hosts>
  ingress_controller_svc_nodePort: <ingress-nginx-controller-nodePort>
```

---

## 📝 Notes

- Any value under `secret` **must be base64 encoded**.
- Do **not** decode base64 values if you're editing them directly in `values.yaml`.
- To get the ingress controller's NodePort, run:
  ```bash
  kubectl get svc -n ingress-nginx
  ```
  and look for the service: `ingress-nginx-controller`.
- Use MySQL image version `>= 8.0.11`.

