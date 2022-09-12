# Basic setup to build

- 0 -> Request from the browser
- 1 -> External service
- 2 -> Relaying to pod with Mongo Express (FRONT for MongoDB manipulation)
- 3 -> Relaying to the internal service
- 4 -> Action on the DB on another pods thanks to the information in `ConfigMap` & `Secret`

![img.png](img.png)


# Starting with DB its secrets and service

- Objectives are

![img_2.png](img_2.png)

- For Base64 encoding

```shell
echo -n <plainText> | base64
```

- Configuration files [deployment](mongodb-deployment.yaml) and [secret](mongodb-secret.yaml)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret # <- Name for reference
type: Opaque # <- Means secret in key-value
data: # <- All the key-value pair stored here in Base64
  mongo-root-username: dXNlcm5hbWU= # <- "username"
  mongo-root-password: cGFzc3dvcmQ= # <- "password"
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-dep
  labels:
    app: mongo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mongo-app
  template:
    metadata:
      labels:
        app: mongo-app
    spec:
      containers:
        - name: mongodb
          image: mongo:5.0.9
          ports:
            - containerPort: 27017
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret # <- reference to an already deployed secret
                  key: mongo-root-username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret # <- reference to an already deployed secret
                  key: mongo-root-password
```

![img_1.png](img_1.png)

- Commands used for both deployment above

```shell
kubectl get all
kubectl apply -f <configFile>
kubeclt get secret
kubectl describe pod <podName>
kubectl get pod --watch
```

- Creating the internal service for the DB

```yaml
<previousDeployment>
--- # <- On the same yaml file but separation for the service
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongo-app # <- The same label as for the pod
  ports:
    - protocol: TCP
      port: 27017 # <- Service port
      targetPort: 27017 # <- Container port
```

- Deploy and Check

```shell
kubectl apply -f <configFile>
kubectl get service
kubectl describe service
```


# Continue on FRONT with configmap and services

- Objectives are

![img_3.png](img_3.png)

- Config map for storing the url DB (= service of MongoDB)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```

- The deployment for mongo-express

```yaml
apiVersion: v1
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
            image: mongo-express:0.54
            ports:
              - containerPort: 8081
            env:
              - name: ME_CONFIG_MONGODB_ADMINUSERNAME
                valueFrom:
                  secretKeyRef:
                    name: mongodb-secret # <- reference to an already deployed secret
                    key: mongo-root-username
              - name: ME_CONFIG_MONGODB_ADMINPASSWORD
                valueFrom:
                  secretKeyRef:
                    name: mongodb-secret # <- reference to an already deployed secret
                    key: mongo-root-password
              - name: ME_CONFIG_MONGODB_SERVER
                valueFrom:
                  configMapKeyRef:
                    name: mongodb-configmap
                    key: database_url # <- ⚠️⚠️⚠️ reference to the configmap
```

- LoadBalancer service opened for Kubernetes external usage

```yaml
<previousDeployment>
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express # <- The same label as for the pod
  type: LoadBalancer # <- ⚠️⚠️⚠️ Characterisation of a LoadBalancer service
  ports:
    - protocol: TCP
      port: 8081 # <- Service port
      targetPort: 8081 # <- Container port
      nodePort: 30000 # <- ⚠️⚠️⚠️ Means ready to be opened to the outside world
```

- To expose the minikube endpoint with the following command

```shell
minikube service mongo-express-service
```

- And this is possible because it's a `LoadBalancer` service with `nodePort`

![img_4.png](img_4.png)