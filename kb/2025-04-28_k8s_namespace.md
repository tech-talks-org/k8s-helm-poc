# Kubernetes Namespace Experiment – Basic Level

## Objective
Understand how to create, use, and isolate resources in different namespaces.

---

## Step 1: Create a namespace

```bash
kubectl create namespace test-namespace
```

Check that it exists:

```bash
kubectl get namespaces
```

---

## Step 2: Deploy something inside the namespace

Example: a simple `nginx` pod.

```bash
kubectl run nginx-test --image=nginx --restart=Never -n test-namespace
```

Verify the pod is present:

```bash
kubectl get pods -n test-namespace
```

---

## Step 3: Try not to see it from the default namespace

If you run:

```bash
kubectl get pods
```

(without specifying `-n`), **you won't see the pod**!
Each namespace isolates its own resources.

---

## Step 4: Delete only the namespace

```bash
kubectl delete namespace test-namespace
```

**Note:** Kubernetes will automatically destroy all resources (pods, services, configmaps, etc.) associated with that namespace.

---

## (Optional) Step 5: Use a temporary namespace context

To avoid specifying `-n` every time:

```bash
kubectl config set-context --current --namespace=test-namespace
```

From that moment on, all commands will default to `test-namespace` without needing to specify it explicitly.

---

## What you learn by doing this
- How to isolate resources in Kubernetes
- How to switch views between different environments
- How to easily clean up groups of resources

---

### Bonus
After completing this experiment, you can try:
- Creating **two different namespaces**
- Deploying **two apps with the same names**
- Verifying that **they don't interfere** thanks to namespace isolation.


# Kubernetes Helm + Namespace Multi-Environment Experiment

## Objective
Deploy the same application in different namespaces (`tim` and `vodafone`) using Helm, demonstrating environment separation.

---

## Step 1: Install the backend application into the `tim` namespace

```bash
helm install my-backend ./k8s-helm/backend -f ./k8s-helm/backend/values-test.yaml --namespace tim --create-namespace
```

Check the pods:

```bash
kubectl get pods -n tim
```

Expected output:

| NAME | READY | STATUS | RESTARTS | AGE |
|:---|:---|:---|:---|:---|
| my-backend-xxx | 1/1 | Running | 0 | Xs |
| my-backend-redis-master-0 | 1/1 | Running | 0 | Xs |
| my-backend-redis-replicas-0 | 1/1 | Running | 0 | Xs |

---

## Step 2: Install the same backend application into the `vodafone` namespace

```bash
helm install my-backend ./k8s-helm/backend -f ./k8s-helm/backend/values-test.yaml --namespace vodafone --create-namespace
```

Check the pods:

```bash
kubectl get pods -n vodafone
```

Expected output:

| NAME | READY | STATUS | RESTARTS | AGE |
|:---|:---|:---|:---|:---|
| my-backend-xxx | 1/1 | Running | 0 | Xs |
| my-backend-redis-master-0 | 1/1 | Running | 0 | Xs |
| my-backend-redis-replicas-0 | 1/1 | Running | 0 | Xs |

---

## Step 3: Verify deployments

List the Helm releases in each namespace:

```bash
helm list -n tim
helm list -n vodafone
```

Both should show a release named `my-backend`, deployed in their respective namespaces.

Check all pods across namespaces:

```bash
kubectl get pods -A
```

You should see pods running in both `tim` and `vodafone` namespaces.

---

## Step 4: Access the application

To access the application locally:

```bash
export POD_NAME=$(kubectl get pods --namespace tim -l "app.kubernetes.io/name=backend,app.kubernetes.io/instance=my-backend" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace tim $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
kubectl --namespace tim port-forward $POD_NAME 8080:$CONTAINER_PORT
```

Repeat similarly for `vodafone` by changing the namespace.

---

## Step 5: Cleanup

```bash
helm uninstall my-backend -n tim
helm uninstall my-backend -n vodafone
kubectl delete namespace tim
kubectl delete namespace vodafone
```

---

## What you learn by doing this
- How to deploy the same Helm chart in multiple isolated environments
- How to work with namespaces and releases simultaneously
- How to manage multi-environment deployments cleanly

---

## Extra: View all pods across all namespaces

```bash
kubectl get pods --all-namespaces
```

or shorthand:

```bash
kubectl get pods -A
```
This will show you all pods running in all namespaces, allowing you to verify that your deployments are isolated correctly.