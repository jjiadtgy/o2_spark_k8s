# Pre-requisite

```shell
helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator
minikube start --cpus 4 --memory 8192
```

# Some commands

```shell
kubectl create namespace spark-appns
kubectl config set-context --current --namespace spark-appns
helm install myrelease spark-operator/spark-operator --namespace james-spark-opns --create-namespace --set sparkJobNamespace=james-spark-appns
kubectl apply -f /Users/jamesjiang/IdeaProjects/o2_spark_k8s/sparkK8SExample/pi.yaml
minikube delete --all
```