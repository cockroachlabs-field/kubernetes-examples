# Kubernetes Demo
The following steps will install Kubernetes locally and configure a 3 node insecure CockroachDB cluster using Helm.  For more information please refer to our official documentation [here](https://www.cockroachlabs.com/docs/stable/orchestrate-a-local-cluster-with-kubernetes-insecure.html).

## Prerequisites
Before getting started make sure the following tools are installed.

* [VirtualBox](https://www.virtualbox.org) - VirtualBox is a general-purpose full virtualizer for x86 hardware
* [Homebrew](https://brew.sh) - a package manager for macOS
* [Kubectl](https://kubernetes.io/docs/setup/minikube/) - Kubernetes command-line tool
* [Minikube](https://kubernetes.io/docs/setup/minikube/) - Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) 
* [Helm + Tiller](https://helm.sh/) - the package manager for Kubernetes

### VirtualBox
Download and install VirtualBox from here: https://www.virtualbox.org/wiki/Downloads

### Homebrew
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
reference: https://brew.sh/

### Kubectl
```bash
brew install kubernetes-cli
```
reference: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-homebrew-on-macos

### Minikube
Insttall Minikube...
```bash
brew cask install minikube
```

then start the service...
```bash
minikube start --cpus 4 --memory 8192
```
reference: https://kubernetes.io/docs/tasks/tools/install-minikube/#macos


### Helm and Tiller
Install Helm...
```bash
brew install kubernetes-helm
```

then install `tiller` by running the following...
```bash
helm init
```

reference: https://helm.sh/docs/using_helm/#from-homebrew-macos, https://helm.sh/docs/using_helm/#installing-tiller

## Installing and Running CockroachDB
Update your `helm` repo to ensure you are pulling the latest version of CockroachDB
```bash
helm repo update
```

Install CockroachDB using helm.  For the curious... https://github.com/helm/charts/tree/master/stable/cockroachdb
```bash
helm install --name k8demo stable/cockroachdb
```

Check for running pods
```bash
kubectl get pods
```

## Monitor CockroachDB
In a new tab, run port forwarding so you can access the CockroachDB UI
```bash
kubectl port-forward k8demo-cockroachdb-0 8080
```

Open the CockroachDB UI: http://localhost:8080

## Create and Run a Workload

Create a the `bank` workload on the cluster...
```bash
kubectl run workload-init -it --image=cockroachdb/cockroach:v19.1.1 --rm --restart=Never -- workload init bank 'postgresql://root@k8demo-cockroachdb-public:26257?sslmode=disable'
```

then run the workload...
```bash
kubectl run workload-run -it --image=cockroachdb/cockroach:v19.1.1 --rm --restart=Never -- workload run bank --duration=10m 'postgresql://root@k8demo-cockroachdb-public:26257?sslmode=disable'
```
reference: See https://www.cockroachlabs.com/docs/stable/cockroach-workload.html#bank-workload

## Kill a Node
```bash
kubectl delete pod k8demo-cockroachdb-2
```

## Add a Node
```bash
kubectl scale statefulset k8demo-cockroachdb --replicas=4
```

## Appendix

### Check Logs
```bash
kubectl logs k8demo-cockroachdb-2
```

### Open SQL Client
```bash
kubectl run sql-client -it --image=cockroachdb/cockroach:v19.1.1 --rm --restart=Never -- sql --insecure --host=k8demo-cockroachdb-public
```
