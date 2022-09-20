# Terminology

- `Core` will be in milli-cores
  - For example `200m` means `0,2 core(s)`
- `Memory` will be in `Mebi-bytes`
  - For example `32Mi` means `≈ 32Mb`

- `Request` means cores OR resources that container will sure to get
  - Otherwise, will not be started not be started
- `Limit` means the maximum number of resources that a container can get
  - Above the threshold in core the container will be virtually sub-performant (BAD ⚠️)
  - Above the threshold in memory the container will be terminated


# Example of usage

- In container settings `requests` and `limits`

```yaml
<someConf>
--- # <- Some configuration just above
containers:
  - name: mongodb
    image: mongo:5.0.9
    resources: # <---- ⚠️ THE IMPORTANT SECTION
      requests:
        memory: "32Mi"
        cpu: "200m"
      limits:
        memory: "64Mi"
        cpu: "250m"
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

- In `ResourceQuotas` the `requests` & `limits` are for all combined containers scheduled in the namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo
  namespace: james-testing-rq
spec:
  hard:
    requests.cpu: 500
    requests.memory: 100Mi
    limits.cpu: 700
    limits.memory: 500Mi
```

- In limit range `LimitRange`
  - If `max` set & `default` not set then the `default` limit is the set `max`
  - If `min` set & `defaultRequest` not set then `defaultRequest` limit is the set `min`

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: demo
  namespace: james-testing-rq
spec:
  limits:
    - default: # <- Default `limits` for all containers if not specified
        cpu: 600
        memory: 100Mi
      defaultRequest: # <- Default `requests` for all containers if not specified
        cpu: 100
        memory: 50Mi
      max: # <- Maximum `limits` for containers
        cpu: 1000
        memory: 200Mi
      min: # <- Minimum `limits` for containers
        cpu: 10
        memory: 10Mi
      type: Container
```


# Overcommitment cases

- It means when an issue arrive, on an over-used node then kill in order of
  - If `Usage` > `Limit` then kill these pod first
  - If `Limit` > `Usage` > `Request` then kill these pod second
  - If `Limit` > `Request` > `Usage` then kill according their priorities
  - First-in-first-served + Neither contention nor changes to quota will affect already created resources
