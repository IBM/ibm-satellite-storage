# IBM block storage Container Storage Interface driver

IBM block storage CSI driver is based on an open-source IBM project, included as a part of IBM Storage orchestration for containers. IBM Storage orchestration for containers enables enterprises to implement a modern container-driven hybrid multicloud environment that can reduce IT costs and enhance business agility, while continuing to derive value from existing systems.

For full release notes, compatiblity, installation, and user information, see the [IBM block storage CSI driver documentation](https://www.ibm.com/docs/en/stg-block-csi-driver/1.11.1).

Supported IBM storage systems:
  - IBM Spectrum Virtualize Family including IBM SAN Volume Controller (SVC) and IBM FlashSystem® family members built with IBM Spectrum® Virtualize (including FlashSystem 5xxx, 7xxx, 9xxx))
  - IBM FlashSystem A9000 and A9000R
  - IBM DS8000 Family
  
## Prerequisites

Review the [compatibility and requirements documentation](https://www.ibm.com/docs/en/stg-block-csi-driver/1.11.1?topic=installing-compatibility-requirements). 

**Important:** Be sure to complete all prerequisite and installation steps before assigning hosts to your location. Do not create a Kubernetes cluster. This is done through Satellite.

## IBM block storage CSI driver parameters and how to retrieve them
Retrieve all parameters required by this template.

```sh
ibmcloud sat storage template get --name ibm-system-storage-block-csi-driver --version 1.11.1
```

 **IBM block storage CSI driver parameters**
 
 For more information about the fields and examples, review the [Creating a StorageClass documentation](https://www.ibm.com/docs/en/stg-block-csi-driver/1.11.1?topic=configuring-creating-storageclass)
| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `namespace` | Optional | The namespace where you want to create the deployment. | `default` |
| `name` | Required | The name of the storage class name that is created. | N/A |
| `space-efficiency` | Optional | The space efficiency of the volume that is created. | N/A |
| `pool` | Required | Name of an existing pool on the storage system where you want to create the volume. | N/A |
| `secret-name` | Required | The name of your existing Kubernetes secret. | N/A |
| `secret-namespace` | Required | The namespace that your Kubernetes secret is in. (can be different then the deployment namespace) | N/A |
| `fstype` | Optional | The file system type. | `ext4` |
| `prefix` | Optional | The prefix name of the volume that is created. | N/A |
| `VolumeExpansion` | Optional |  Specify `true` to allow volume expansion or `false` to disallow volume expansion. | `false` |


## Creating the IBM block storage CSI driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <config-name> --template-name ibm-system-storage-block-csi-driver --template-version 1.11.1 -p "namespace=<namespace>" -p "name=<sc-name>" -p "space-efficiency=<space-efficiency>" -p "pool=<pool>" -p "secret-name=<secret-name>" -p "secret-namespace=<secret-namespace>" -p "fstype=<fstype>" -p "prefix=<prefix>" -p "VolumeExpansion=<VolumeExpansion>"

```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name <assignment-name> --group <cluster-group-name> --config <config-name>
```

## Verifying your IBM block storage CSI driver storage configuration is assigned to your clusters

Run the following commands to verify that the driver pods are running.

```bash
$ kubectl get all -n <namespace>  -l csi
NAME                             READY   STATUS    RESTARTS   AGE
pod/ibm-block-csi-controller-0   6/6     Running   0          9m36s
pod/ibm-block-csi-node-jvmvh     3/3     Running   0          9m36s
pod/ibm-block-csi-node-tsppw     3/3     Running   0          9m36s

NAME                                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/ibm-block-csi-node   2         2         2       2            2           <none>          9m36s

NAME                                        READY   AGE
statefulset.apps/ibm-block-csi-controller   1/1     9m36s
```
## Removing your assignment

When you remove the storage assignment from your clusters, the following resources are removed.
 - All pods, daemonsets and statefulsets from the deployment.
 - All objects that are created manually will remain on the cluster. For example: app pods, PVC, PVs that you created manually, must be manually removed.
  
```sh
ibmcloud sat storage assignment rm --assignment <assignment-name>
```

## Removing your configuration

After the assignment has been removed it is safe to remove the configuration.
Note that removing the configuration is not mandatory. You can reuse your storage configuration on other clusters.
```sh
ibmcloud sat storage config rm --config <config-name>
```

## Troubleshooting, support, and logs

[Troubleshooting documentation](https://www.ibm.com/docs/en/stg-block-csi-driver/1.11.1?topic=troubleshooting)

## References

- [IBM block storage CSI driver documentation](https://www.ibm.com/docs/en/stg-block-csi-driver)