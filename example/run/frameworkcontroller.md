# Run FrameworkController

- Ensure at most one instance of FrameworkController is run for a single k8s cluster.
- For the full FrameworkController configuration, see
 [Config Usage](../../pkg/apis/frameworkcontroller/v1/config.go) and [Config Example](../../example/config/default/frameworkcontroller.yaml).

## Run by a Kubernetes StatefulSet

- Using official image to demonstrate this example.

```shell
kubectl create -f frameworkcontroller.yaml
```

frameworkcontroller.yaml:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: frameworkcontroller
  namespace: default
spec:
  serviceName: frameworkcontroller
  selector:
    matchLabels:
      app: frameworkcontroller
  replicas: 1
  template:
    metadata:
      labels:
        app: frameworkcontroller
    spec:
      # Need to grant permission if target cluster enforces authorization, such as RBAC:
      #   kubectl create serviceaccount frameworkcontroller --namespace default
      #   kubectl create clusterrolebinding frameworkcontroller --clusterrole=cluster-admin --user=system:serviceaccount:default:frameworkcontroller
      # See k8s service account and authorization:
      #   https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account
      #   https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules
      #serviceAccountName: frameworkcontroller
      containers:
      - name: frameworkcontroller
        image: frameworkcontroller/frameworkcontroller
        # No need to specify KUBE_APISERVER_ADDRESS or KUBECONFIG if the target cluster
        # to control is the cluster running the StatefulSet.
        # See k8s inClusterConfig:
        #   https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#accessing-the-api-from-a-pod
        #env:
        #- name: KUBE_APISERVER_ADDRESS
        #  value: {http[s]://host:port}
        #- name: KUBECONFIG
        #  value: {Pod Local KubeConfig File Path}
```

## Run by a Docker Container

- Using official image to demonstrate this example.

If you have an insecure ApiServer address (can be got from [Insecure ApiServer](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/#api-server-ports-and-ips) or [kubectl proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#using-kubectl-proxy)) which does not enforce authentication, you only need to provide the address:
```shell
docker run -e KUBE_APISERVER_ADDRESS={http[s]://host:port} frameworkcontroller/frameworkcontroller
```

Otherwise, you only need to provide your [KubeConfig File](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#explore-the-home-kube-directory)  which inlines or refers the [ApiServer Credential Files](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/#transport-security):
```shell
docker run -e KUBECONFIG=/mnt/.kube/config -v {Host Local KubeConfig File Path}:/mnt/.kube/config -v {Host Local ApiServer Credential File Path}:{Container Local ApiServer Credential File Path} frameworkcontroller/frameworkcontroller
```
For example, if the k8s cluster is created by [Minikube](https://kubernetes.io/docs/setup/minikube):
```shell
docker run -e KUBECONFIG=/mnt/.kube/config -v ${HOME}/.kube/config:/mnt/.kube/config -v ${HOME}/.minikube:${HOME}/.minikube frameworkcontroller/frameworkcontroller
```

## Run by a OS Process

- Using local built binary distribution to demonstrate this example.
- Prerequisite: Clone the source code into `${GOPATH}/src/github.com/microsoft/frameworkcontroller` and build the binary distribution by the [go-build.sh](../../build/frameworkcontroller/go-build.sh).

If you have an insecure ApiServer address (can be got from [Insecure ApiServer](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/#api-server-ports-and-ips) or [kubectl proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#using-kubectl-proxy)) which does not enforce authentication, you only need to provide the address:
```shell
KUBE_APISERVER_ADDRESS={http[s]://host:port} ./dist/frameworkcontroller/start.sh
```

Otherwise, you only need to provide your [KubeConfig File](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#explore-the-home-kube-directory) which inlines or refers the [ApiServer Credential Files](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/#transport-security):
```shell
KUBECONFIG={Process Local KubeConfig File Path} ./dist/frameworkcontroller/start.sh
```
For example:
```shell
KUBECONFIG=${HOME}/.kube/config ./dist/frameworkcontroller/start.sh
```
