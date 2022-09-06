# RBAC base concepts

- Role based access control, it involves
  - `User` or `Service account`
    - the first is used for connection from external (actual user)
    - the second is used internally between pods
    - they both lived in a namespace level
  - `Role` & `RoleBinding`
    - the first is to define authorization at the namespace level
    - the second is to link a `Role` to a `User` or `Service account` at the namespace level
  - `ClusterRole` & `ClusterRoleBinding`
    - both idem but at a cluster level (between namespaces)

| `User` & `Service account` with some `Roles` | `RoleBinding`            |
|----------------------------------------------|--------------------------|
| ![img.png](img.png)                          | ![img_1.png](img_1.png)  |


# To properly add a new user on the cluster

- Create a certificate request `bob.csr` with `OpenSSL`

| Generate Key            | Generate Certificate    |
|-------------------------|-------------------------|
| ![img_2.png](img_2.png) | ![img_3.png](img_3.png) |

- Sign certificate request with Kubernetes CA files `ca.crt` & `ca.key` to have `bob.crt`
  - on the Master node `/etc/kubernetes/pki` folder
  - on the host PC for minikube `$HOME/.minikube`
  - Managed Kubernetes use OAuth instead (≈ No need all the steps above)

![img_4.png](img_4.png)

- `.kube/config` contains
  - Clusters 👉 with API & CAs
  - Users 👉 `.crt` token usable for authentication
  - Context 👉 binding Clusters to Users

| Abstract                | Actual                  |
|-------------------------|-------------------------|
| ![img_5.png](img_5.png) | ![img_6.png](img_6.png) |

- So we need to add these 3 things in our `.kube/config` to connect to the cluster

| Setup cluster             ||
|---------------------------|------------------------------|
| ![img_7.png](img_7.png)   | ![img_8.png](img_8.png)      |

| Setup user                ||
|---------------------------|------------------------------|
| ![img_9.png](img_9.png)   |![img_10.png](img_10.png)|

| Setup context             ||
|---------------------------|------------------------------|
| ![img_11.png](img_11.png) |![img_12.png](img_12.png)|

- Switch the current context `kubectl config use-context dev`


# Roles & Roles binding

- Allowing `kubectl <verb>` to an external user or group

| Role to create            | Role binding to create    |
|---------------------------|---------------------------|
| ![img_13.png](img_13.png) | ![img_14.png](img_14.png) |

- Commands to apply these
  - `kubectl create ns shopping` (if the namespace is not existing yet)
  - `kubectl -n shopping apply -f <role.yaml>`
  - `kubectl -n shopping apply -f <rolebinding.yaml>`


# Service Account

- Internal inside Kubernetes "User"

![img_15.png](img_15.png)

- Then Pods can use this identity `ServiceAccount` to 

![img_16.png](img_16.png)

- If you check inside the Pod, you can exactly see the same idea of signed certificated we built for external user
  - So applications inside the pods can authenticate to the cluster

![img_18.png](img_18.png)

- For the `ServiceAccount` (≈ Our external user) we need to define `Role` & `RoleBinding`

| Role to create            | Role binding to create    |
|---------------------------|---------------------------|
| ![img_17.png](img_17.png) | ![img_19.png](img_19.png) |