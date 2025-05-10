# Deploy MongoDB and Mongo Express to Local Kubernetes Cluster

## Technologies Used

* Kubernetes
* Docker
* MongoDB
* Mongo Express

## Step 1: Install Minikube (via Homebrew)

```bash
brew install minikube
```

## Step 2: Start Minikube

Make sure Docker Desktop is running before executing:

```bash
minikube start
```

## Step 3: Create the YAML Files

### 1. `mongo-secret.yaml`

Contains the MongoDB credentials.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU=
  mongo-root-password: cGFzc3dvcmQ=
```

### 2. `mongo.yaml`

Deploys MongoDB using the secret for credentials.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo
          ports:
            - containerPort: 27017
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-root-username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-root-password


```

### 3. `mongo-service.yaml`

Exposes the MongoDB service internally.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

### 4. `mongoexp-config.yaml`

Defines environment variables for Mongo Express.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-configmap
data:
  database_url: mongodb-service
```

### 5. `mongoexp.yaml`

Deploys Mongo Express.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
        - name: mongo-express
          image: mongo-express
          ports:
            - containerPort: 8081
          env:
            - name: ME_CONFIG_MONGODB_ADMINUSERNAME
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-root-username
            - name: ME_CONFIG_MONGODB_ADMINPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-root-password
            - name: ME_CONFIG_MONGODB_SERVER
              valueFrom:
                configMapKeyRef:
                  name: mongo-configmap
                  key: database_url
```

### 6. `mongoexp-service.yaml`

Exposes Mongo Express as a NodePort.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000

```

## Step 4: Apply All Configurations

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo.yaml
kubectl apply -f mongo-service.yaml
kubectl apply -f mongoexp-config.yaml
kubectl apply -f mongoexp.yaml
kubectl apply -f mongoexp-service.yaml
```
