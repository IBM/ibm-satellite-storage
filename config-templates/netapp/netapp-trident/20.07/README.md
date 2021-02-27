# NetApp Trident

Trident tightly integrates with Kubernetes to allow users to request and manage persistent volumes using native Kubernetes interfaces and constructs. It’s designed to work in such a way that users can take advantage of the underlying capabilities of the ONTAP storage infrastructure without having to know anything about it.
This template just deploy Trident, the universal operator, on your target cluster and makes the cluster ready for Ontap-NAS and Ontap-SAN driver deployment.
## Prerequisites
N/A


## NetApp Trident Driver parameters & how to retrieve them
N/A


## Default storage classes
N/A

## Creating the NetApp Trident storage configuration

**Example `sat storage config create` command**

```
ibmcloud sat storage config create --name 'trident-config' --template-name 'netapp-trident' --template-version '20.07'
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name 'trident-operator' --group <group name> --config 'trident-config'
```

## Verifying your NetApp Trident configuration is assigned to your clusters.

```
oc -n trident get all
```


**Example output**

```
$ oc -n trident get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/trident-csi-5vzxq                   2/2     Running   0          61s
pod/trident-csi-7f999bfb96-tz4n6        6/6     Running   0          61s
pod/trident-csi-cr9xh                   2/2     Running   0          61s
pod/trident-csi-psvn4                   2/2     Running   0          61s
pod/trident-operator-794f74cd4b-dff85   1/1     Running   0          71s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
service/trident-csi   ClusterIP   172.21.124.112   <none>        34571/TCP,9220/TCP   61s

NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                     AGE
daemonset.apps/trident-csi   3         3         3       3            3           kubernetes.io/arch=amd64,kubernetes.io/os=linux   62s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/trident-csi        1/1     1            1           62s
deployment.apps/trident-operator   1/1     1            1           73s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/trident-csi-7f999bfb96        1         1         1       62s
replicaset.apps/trident-operator-794f74cd4b   1         1         1       73s
```

## Troubleshooting

In case the required resource are getting create check the Trident Opertor log
```
oc -n trident logs <trident operator pod name>
```

## Reference

- https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-san/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html
