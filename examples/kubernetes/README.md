# Ceph on Halcyon-Kubernetes


### Deploy Ceph Components

With the secrets created, you can now deploy ceph.

```
kubectl get nodes -L kubeadm.alpha.kubernetes.io/role --no-headers | awk '$NF ~ /^<none>/ { print $1}' | while read NODE ; do
kubectl label node $NODE --overwrite node-type=storage
kubectl label node $NODE --overwrite openstack-control-plane=enabled
done
kubectl create namespace ceph
kubectl create \
-f ceph-bootstrap-secrets-job.yaml \
-f ceph-mds-v1-dp.yaml \
-f ceph-mon-v1-svc.yaml \
-f ceph-mon-v1-dp.yaml \
-f ceph-mon-check-v1-dp.yaml \
-f ceph-osd-v1-ds.yaml \
-f ceph-storage.yaml \
--namespace=ceph
kubectl get  --namespace=ceph pods
```

Your cluster should now look something like this.

```
$ kubectl get all --namespace=ceph
NAME                   DESIRED      CURRENT       AGE
ceph-mds               1            1             24s
ceph-mon-check         1            1             24s
NAME                   CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
ceph-mon               None         <none>        6789/TCP   24s
NAME                   READY        STATUS        RESTARTS   AGE
ceph-mds-6kz0n         0/1          Pending       0          24s
ceph-mon-check-deek9   1/1          Running       0          24s
```

### Label your storage nodes

You must label your storage nodes in order to run Ceph pods on them.

```
kubectl label node <nodename> node-type-storage
```

If you want all nodes in your Kubernetes cluster to be a part of your Ceph cluster, label them all.

```
kubectl label nodes node-type=storage --all
```

Eventually all pods will be running, including a mon and osd per every labeled node.

```
$ kubectl get pods --namespace=ceph
NAME                   READY     STATUS    RESTARTS   AGE
ceph-mds-6kz0n         1/1       Running   0          4m
ceph-mon-8wxmd         1/1       Running   2          2m
ceph-mon-c8pd0         1/1       Running   1          2m
ceph-mon-cbno2         1/1       Running   1          2m
ceph-mon-check-deek9   1/1       Running   0          4m
ceph-mon-f9yvj         1/1       Running   1          2m
ceph-osd-3zljh         1/1       Running   2          2m
ceph-osd-d44er         1/1       Running   2          2m
ceph-osd-ieio7         1/1       Running   2          2m
ceph-osd-j1gyd         1/1       Running   2          2m
```
