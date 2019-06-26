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
Install Minikube...
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

Your output will start with:
```bash
NAME:   k8demo
LAST DEPLOYED: <date>
NAMESPACE: default
STATUS: DEPLOYED
```

Check for running pods
```bash
kubectl get pods
```
You will see something like the following if run correctly:

```bash
NAME                            READY     STATUS      RESTARTS   AGE
k8demo-cockroachdb-0            1/1       Running     0          16s
k8demo-cockroachdb-1            1/1       Running     0          16s
k8demo-cockroachdb-2            1/1       Running     0          16s
k8demo-cockroachdb-init-lmz9d   0/1       Completed   0          16s
```

## Monitor CockroachDB
In a new tab, run port forwarding so you can access the CockroachDB UI
```bash
kubectl port-forward k8demo-cockroachdb-0 8080
```

If you've run this correctly, the output should look like this:

```bash
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

With additional lines showing when you go to port 8080 in the browser.

This will take over your terminal window, and you'll need to open a new tab to run the next shell command.

Open the CockroachDB UI in your browser: [http://localhost:8080](http://localhost:8080)

## Create and Run a Workload

Create a the `bank` workload on the cluster...
```bash
kubectl run workload-init -it --image=cockroachdb/cockroach:v19.1.1 --rm --restart=Never -- workload init bank 'postgresql://root@k8demo-cockroachdb-public:26257?sslmode=disable'
```

If run correctly, there will be no output.

then run the workload...
```bash
kubectl run workload-run -it --image=cockroachdb/cockroach:v19.1.1 --rm --restart=Never -- workload run bank --duration=10m 'postgresql://root@k8demo-cockroachdb-public:26257?sslmode=disable'
```
reference: See https://www.cockroachlabs.com/docs/stable/cockroach-workload.html#bank-workload

This will take over your terminal for 10 minutes, so you'll need to open a new tab for the next command.

## Kill a Node
```bash
kubectl delete pod k8demo-cockroachdb-2
```

If this command works, then `pod "k8demo-cockroachdb-2" deleted` will be displayed.

Note that on the overview page of your dashboard, at [http://localhost:8080/#/overview/list](http://localhost:8080/#/overview/list), and the node will be automatically be restarted by Kubernetes shortly.

If you keep an eye on the admin console, you will see the node restart within a few minutes.


## Add a Node
```bash
kubectl scale statefulset k8demo-cockroachdb --replicas=4
```

If this runs correctly, the shell will print `statefulset.apps "k8demo-cockroachdb" scaled`.

After a few seconds, you can refresh the admin UI to see the new node.

## Appendix

### Check Logs
```bash
kubectl logs k8demo-cockroachdb-2 | less
```

Use the `q` key to exit.

### Open SQL Client
```bash
kubectl run sql-client -it --image=cockroachdb/cockroach:v19.1.1 --rm --restart=Never -- sql --insecure --host=k8demo-cockroachdb-public
```

Use `ctrl-d` or `exit` to exit the SQL shell.

### Cleaning up

When you're ready to clean up your test environment, run the following:

Stop the kubectl process (`control-c` will work, as will closing the terminal window).

Run the following to purge the installation of CockroachDB with helm:

```bash
helm del --purge k8demo
```

Stop the minikube environment:

```bash
minikube stop
```

... and/or delete it with

```bash
minikube delete
```

