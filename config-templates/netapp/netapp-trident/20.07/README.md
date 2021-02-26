# Netapp Trident

Trident integrates natively with Kubernetes and its Persistent Volume framework to seamlessly provision and manage volumes from systems running any combination of NetApp’s ONTAP

## Prerequisites



## Netapp Trident Driver parameters & how to retrieve them



## Default storage classes


## Creating the Netapp Trident storage configuration


**Example `sat storage config create` command**

```
ibmcloud sat storage config create --name "trident-conf" --template-name "netapp-trident" --template-version "20.07"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name "trident-operator --group <group name> --config "trident-conf"
```

## Verifying your Netapp Trident configuration is assigned to your clusters.

```
oc get pods -n trident 
```


**Example output**

Provide an example output screen of running driver pods and storage classes that are deployed by your configuration.


## Troubleshooting

Provide troubleshooting steps for any known issues.


## Reference

- https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-san/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html
