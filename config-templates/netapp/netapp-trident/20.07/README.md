# Netapp Trident

Trident tightly integrates with Kubernetes to allow users to request and manage persistent volumes using native Kubernetes interfaces and constructs. It’s designed to work in such a way that users can take advantage of the underlying capabilities of the ONTAP storage infrastructure without having to know anything about it.
This template just deploy Trident, the universal operator, on your target cluster and makes the cluster ready for Ontap-NAS and Ontap-SAN driver deployment.
## Prerequisites
N/A


## Netapp Trident Driver parameters & how to retrieve them
N/A


## Default storage classes
N/A

## Creating the Netapp Trident storage configuration

**Example `sat storage config create` command**

```
ibmcloud sat storage config create --name 'trident-config' --template-name 'netapp-trident' --template-version '20.07'
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name 'trident-operator' --group <group name> --config 'trident-config'
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
