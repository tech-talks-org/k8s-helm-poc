# PoC: Kubernetes + Helm in Dev Container

This project demonstrates how to set up a **Node.js backend** application running in **Minikube** inside a **Dev Container** using **Helm**.

The goal is to test the execution of a Kubernetes app in a local development environment without relying on external infrastructure.

## Project Structure

```
k8s-helm-devcontainer-poc/
│── .devcontainer/
│   ├── devcontainer.json
│── backend/
│   ├── index.js
│   ├── package.json
│   ├── Dockerfile
│── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
```

### Dev Container Setup

The project uses Dev Containers to provide a preconfigured development environment.

We used the **VS Code Dev Container wizard** to set up the environment, selecting the necessary features.

The `.devcontainer/devcontainer.json` file includes:

- A **Node.js development environment**
- The **Docker-in-Docker** feature
- The **kubectl, Helm, and Minikube** feature

### Backend

The backend is a simple **Node.js** application running in an Express server. It responds to HTTP requests and will be deployed inside a Kubernetes cluster.

### Kubernetes Configuration

The Kubernetes configuration includes:

- A **Deployment** that runs the backend container
- A **Service** that exposes the backend within the cluster

## Running the PoC

### 1. Start Minikube
Run only the first time:

```sh
minikube start --driver=docker
```

### 2. Switch to Minikube's Docker daemon

When you use Minikube with the Docker driver, it creates a virtual environment with its own Docker daemon inside the Minikube VM. Connecting to Minikube’s Docker daemon allows you to build and run Docker images directly inside Minikube’s environment, avoiding the need to push images to an external registry.

```sh
eval $(minikube docker-env)
```

### 3. Build the Docker image for the backend

```sh
docker build -t backend:latest ./backend
```

### 4. Apply Kubernetes configurations

```sh
kubectl apply -f k8s/
```

### 5. Check Pod status

```sh
kubectl get pods
```

Expected output:

```
NAME                       READY   STATUS    RESTARTS   AGE
backend-55978757c7-z994t   1/1     Running   0          12s
```

### 6. Port-forward to test the API

```sh
kubectl port-forward service/backend-service 8080:80
```

Now test the endpoint in a browser or with `curl`:

```sh
curl http://localhost:8080/
```

Expected response:

```
Hello from Node.js backend running in Kubernetes!
```

## Helm Integration

### 1. Create the helm folder 

Use the command `helm create backend`

Edit the chart to suit your app (e.g., update values.yaml, set image name to backend:latest).

### 3. Install the chart

`helm install my-backend ./k8s-helm/backend`

If the release already exists:

`helm upgrade my-backend ./k8s-helm/backend`

### 4. Check the running pods

`kubectl get pods`

### 5. Port-forward via Helm label selectors

```
export POD_NAME=$(kubectl get pods -l "app.kubernetes.io/name=backend,app.kubernetes.io/instance=my-backend" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
kubectl port-forward $POD_NAME 8080:$CONTAINER_PORT
```
Then visit: http://localhost:8080






## Troubleshooting

### `ImagePullBackOff` Error

If Kubernetes cannot find the image, ensure you have built it in the Minikube context:

```sh
eval $(minikube docker-env)
docker build -t backend:latest ./backend
kubectl delete pod --all
kubectl apply -f k8s/
```

### **`connection refused` Error in Minikube**

If Minikube does not start correctly:

```sh
minikube delete
minikube start --driver=docker
```

## Next Steps

- Helm integration
