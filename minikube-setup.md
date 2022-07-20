# Setup minikube for Spark

[Install on MacOS](https://minikube.sigs.k8s.io/docs/start/)

```shell
minikube start --cpus 4 --memory 8192 
```


# Key concepts

[Key concepts of Kubernetes](https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e)

- Deployment (In CMD or Config file) > ReplicaSet (Managed the desired number of Pods and Replication for Horizontal scaling) > Pods
- Pods are on Nodes and Run one container in general
- Nodes are directed by Master
- Namespace means sub-cluster of the whole cluster


[Service account explained](https://medium.com/the-programmer/working-with-service-account-in-kubernetes-df129cb4d1cc)

- It's used to authenticate pods to access Kubernetes cluster


# First Spark interaction on Minikube

[Spark Kube](https://jaceklaskowski.github.io/spark-kubernetes-book/demo/spark-shell-on-minikube/)

```shell
$SPARK_HOME/bin/docker-image-tool.sh -m -t my-tag build
```

```shell
./bin/spark-shell \
  --master k8s://https://127.0.0.1:55188 \
  --conf spark.kubernetes.container.image=spark:my-tag \
  --conf spark.kubernetes.namespace=default \
  --verbose
```