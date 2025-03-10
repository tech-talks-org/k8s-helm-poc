# PoC: Kubernetes + Dev Containers + Minikube

## Descrizione
Questo progetto dimostra come configurare un'applicazione backend Node.js eseguibile in **Minikube** all'interno di un **Dev Container**.

L'obiettivo è testare l'esecuzione di un'app Kubernetes in un ambiente di sviluppo locale senza dover dipendere da infrastrutture esterne.

---

## Struttura del Progetto
```
poc-k8s-devcontainer/
│── backend/
│   ├── index.js
│   ├── package.json
│   ├── Dockerfile
│── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│── .devcontainer/
│   ├── devcontainer.json
│── README.md
```

---

## 1. Setup del Dev Container
Il progetto utilizza Dev Containers per fornire un ambiente di sviluppo preconfigurato.

### **Configurazione in `.devcontainer/devcontainer.json`**
```json
{
  "name": "Node.js",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/kubectl-helm-minikube:1": {}
  }
}
```

---

## 2. Creazione del Backend
### **File `backend/index.js`**
```js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello from Node.js backend running in Kubernetes!');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### **Dockerfile per il backend (`backend/Dockerfile`)**
```Dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

---

## 3. Configurazione Kubernetes
### **Deployment (`k8s/deployment.yaml`)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: backend:latest
          ports:
            - containerPort: 3000
          env:
            - name: PORT
              value: "3000"
```

### **Service (`k8s/service.yaml`)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

---

## 4. Esecuzione del PoC
### **1. Avviare Minikube**
```sh
minikube start --driver=docker
```

Se ci sono errori, resettare Minikube:
```sh
minikube delete
minikube start --driver=docker
```

### **2. Passare al daemon Docker di Minikube**
```sh
eval $(minikube docker-env)
```

### **3. Buildare l'immagine Docker per il backend**
```sh
docker build -t backend:latest ./backend
```

### **4. Applicare le configurazioni Kubernetes**
```sh
kubectl apply -f k8s/
```

### **5. Verificare lo stato del Pod**
```sh
kubectl get pods
```
Dovrebbe restituire un output simile a:
```
NAME                       READY   STATUS    RESTARTS   AGE
backend-55978757c7-z994t   1/1     Running   0          12s
```

### **6. Eseguire il forward della porta per testare l'API**
```sh
kubectl port-forward service/backend-service 8080:80
```
Ora puoi testare l'endpoint in un browser o con `curl`:
```sh
curl http://localhost:8080/
```
Dovresti ricevere la risposta:
```
Hello from Node.js backend running in Kubernetes!
```

---

## 5. Troubleshooting
### **Errore `ImagePullBackOff`**
Se Kubernetes non trova l'immagine, assicurati di aver buildato nel contesto di Minikube:
```sh
eval $(minikube docker-env)
docker build -t backend:latest ./backend
kubectl delete pod --all
kubectl apply -f k8s/
```

### **Errore `connection refused` in Minikube**
Se Minikube non parte correttamente:
```sh
minikube delete
minikube start --driver=docker
```

---

## 6. Conclusioni
Questo PoC mostra come eseguire un backend Node.js in Kubernetes utilizzando un Dev Container per l'ambiente di sviluppo.

**Prossimi passi:**
- Automatizzare il processo con `skaffold`
- Integrare un registry locale per la gestione delle immagini
- Estendere l'architettura con più servizi

🚀 Buon coding!

