## Installing and Running CockroachDB (Secure)

Install CockroachDB using `helm`.  For the curious... https://github.com/helm/charts/tree/master/stable/cockroachdb
```bash
helm install --name k8demo --set Secure.Enabled=true stable/cockroachdb
```

Your output will start with:
```bash
NAME:   k8demo
LAST DEPLOYED: <date>
NAMESPACE: default
STATUS: DEPLOYED
```

Approve `pending` CSRs:
```bash
kubectl certificate approve default.node.k8demo-cockroachdb-0
kubectl certificate approve default.node.k8demo-cockroachdb-1
kubectl certificate approve default.node.k8demo-cockroachdb-2
kubectl certificate approve default.client.root
```

You will see something like the following if each command runs correctly:
```bash
certificatesigningrequest.certificates.k8s.io/default.node.k8demo-cockroachdb-0 approved
```

Add the CockroachDB secure client.  See https://www.cockroachlabs.com/docs/stable/orchestrate-a-local-cluster-with-kubernetes.html#step-3-use-the-built-in-sql-client

```bash
curl https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/client-secure.yaml | sed -e 's/serviceAccountName\: cockroachdb/serviceAccountName\: k8demo-cockroachdb/g' | kubectl create -f -
````

Check for running pods:
```bash
kubectl get pods
```
You will see something like the following if run correctly:

```bash
NAME                            READY   STATUS      RESTARTS   AGE
cockroachdb-client-secure       1/1     Running     0          14s
k8demo-cockroachdb-0            1/1     Running     0          5h51m
k8demo-cockroachdb-1            1/1     Running     0          5h51m
k8demo-cockroachdb-2            1/1     Running     0          5h51m
k8demo-cockroachdb-init-q42qd   0/1     Completed   0          5h51m
```

Use the secure client, `cockroachdb-client-secure`, to create a new user.  This will be used to login to the admin UI.  Username is `roach` and password is `password`.
```bash
kubectl exec -it cockroachdb-client-secure -- ./cockroach sql --execute="CREATE USER roach WITH PASSWORD 'password';" --certs-dir=/cockroach-certs --host=k8demo-cockroachdb-public
```


