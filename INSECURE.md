## Installing and Running CockroachDB (Insecure)

1. Install CockroachDB using `helm`.  For the curious see our Helm Chart [here](https://github.com/helm/charts/tree/master/stable/cockroachdb).
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

2. Check for running pods.
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