# Ingress Setup for Local Development with Minikube inside a Dev Container
This guide explains how to expose a Kubernetes service via Ingress while running Minikube **inside a Dev Container** (Docker-in-Docker setup).

---

## 🚀 Prerequisites

- Minikube running inside the Dev Container
- A deployed service (in our case `my-backend`) using Helm
- Helm chart includes an ingress template (e.g., `templates/ingress.yaml`)
- `kubectl` installed and configured
- `curl` installed on the host machine
- Host machine access to modify `/etc/hosts`

---

## 🧱 Step-by-Step Setup

### 1. Start Minikube

```bash
minikube start
```

You should see confirmation that the cluster and Kubernetes components are running.

---

### 2. Enable the NGINX Ingress Controller

```bash
minikube addons enable ingress
```

Check that the controller is running:

```bash
kubectl get pods -n ingress-nginx
```

---

### 3. Configure Helm Values for Ingress

Make sure your Helm `values-test.yaml` includes the following:

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  hosts:
    - host: test.my-backend.local
      paths:
        - path: /()(.*)
          pathType: Prefix
```

Update the Helm release:

```bash
helm upgrade my-backend ./k8s-helm/backend -f ./k8s-helm/backend/values-test.yaml --namespace vodafone
```

> ⚠️ Note: If the release is not deployed yet, run `helm install` instead.

---

### 4. 🔍 Understanding Ingress Values

Here are the key `values.yaml` entries and what they mean:

```yaml
ingress:
  enabled: true               # Enables the ingress template
  className: nginx            # Specifies the ingress controller class (e.g., nginx, traefik)
  annotations:                # Custom behaviors for the ingress controller
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  hosts:
    - host: test.my-backend.local   # Domain name to route traffic
      paths:
        - path: /()(.*)             # Path pattern (can use regex with rewrite)
          pathType: Prefix          # How to interpret the path (Exact, Prefix, ImplementationSpecific)
  tls: []                     # Can be configured for HTTPS (e.g., secretName + hosts)
```

#### 🔹 Common pathType values:
- `Prefix`: Match if the request path **starts with** the defined path. Most common and reliable.
- `Exact`: Match **only if the path is exactly** the same. Useful for specific endpoints like `/health`.
- `ImplementationSpecific`: Delegates interpretation to the ingress controller (may allow regex). Less portable.

#### 🔹 Annotations (example for NGINX):
- `nginx.ingress.kubernetes.io/rewrite-target`: Used to rewrite the incoming path before forwarding it to the backend service.
- Other common ones include rate limiting, basic auth, and custom headers.

You can adapt this structure depending on your application's routing needs.

---


### 5. Check the Ingress Resource

```bash
kubectl get ingress -n vodafone
```

You should see something like:

```
NAME         CLASS   HOSTS                   ADDRESS         PORTS
my-backend   nginx   test.my-backend.local   192.168.49.2    80
```

---

### 5. Port Forward the Ingress Controller

Since you're inside a Dev Container, Minikube is isolated. You must **port-forward** the NGINX Ingress Controller to access it from your host machine:

```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80
```

This will expose Ingress on `http://localhost:8080`.

---

### 6. Add Host Mapping

Edit your host machine's `/etc/hosts` file and map the hostname:

```text
127.0.0.1  test.my-backend.local
```

---

### 7. Test Access

Open a browser or use curl:

```bash
curl http://test.my-backend.local:8080
```

Expected output:

```text
Hello from Node.js backend running in Kubernetes! - test
```

---

## ✅ Notes & Troubleshooting

- You **must** forward the port from the ingress controller to your host (`8080:80`).
- The `path: /()(.*)` combined with `rewrite-target: /$2` allows dynamic rewriting of all sub-paths.
- The Dev Container doesn't allow editing `/etc/hosts`, so changes must be made on your **host OS**, not inside the container.
- Use `kubectl get svc -n ingress-nginx` if you're unsure about the ingress service name.

---

## 🧼 Clean Up

To stop the forwarding:

```bash
Ctrl + C
```

To uninstall the release:

```bash
helm uninstall my-backend --namespace vodafone
```
