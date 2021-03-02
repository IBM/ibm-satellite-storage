# IBM block storage Container Storage Interface driver

IBM block storage CSI driver is based on an open-source IBM project, included as a part of IBM Storage orchestration for containers. IBM Storage orchestration for containers enables enterprises to implement a modern container-driven hybrid multicloud environment that can reduce IT costs and enhance business agility, while continuing to derive value from existing systems.

For full release notes, compatiblity, installation, and user information, see the IBM block storage CSI driver documentation: https://www.ibm.com/support/knowledgecenter/SSRQ8T_1.4.0/

## Prerequisites

For full prerequisite instructions, see the following documentation, referred to in the introductory paragraph. 
**Installation** > **Compatibility and requirements**.

**Important:** Be sure to complete all prerequisite and installation steps before assigning hosts to your location. Do not create a Kubernetes cluster. This is done through Satellite.

## IBM block storage CSI driver parameters & how to retrieve them



Retrieve all parameters required by this template.

```

ibmcloud sat storage template get --name ibm-csi-block --version 1.4.0

```

 **IBM block storage CSI driver parameters**
| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `namespace` | Optional | Deployment namespace. | `default` |
| `sc-name` | Required | Storage class name to be created | N/A |
| `space-efficiency` | Optional | Space efficiency of volume to be created | N/A |
| `pool` | Required | Pool ID of volume to be created in | N/A |
| `secret-name` | Required | Existing secret name | N/A |
| `secret-namespace` | Required | Existing secret name namespace | N/A |
| `fstype` | Optional | File system type | `ext4` |
| `prefix` | Optional | Prefix name of volume to be created | N/A |
| `VolumeExpansion` | Optional | Is volume expansion allowed? | `false` |


## Creating the IBM block storage CSI driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <config-name> --template-name ibm-csi-block --template-version 1.4.0 -p "namespace=<namespace>" -p "sc-name=<sc-name>" -p "space-efficiency=<space-efficiency>" -p "pool=<pool>" -p "secret-name=<secret-name>" -p "secret-namespace=<secret-namespace>" -p "fstype=<fstype>" -p "prefix=<prefix>" -p "VolumeExpansion=<VolumeExpansion>"

```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name <assignment-name> --group <cluster-group-name> --config <config-name>
```

## Verifying your IBM block storage CSI driver storage configuration is assigned to your clusters

To verify that your configuration is assigned to your cluster. Verify that the driver pods are running.


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

When removing the assignment from my clusters:
 - All pods, daemonset and statefulset from the deployment will be removed.
 - All objects (pvc, pv, pod) that created manually will remain on the cluster until manual deletion.
  
```sh
ibmcloud sat storage assignment rm --assignment <assignment-name>
```

## Removing your configuration

After the assignment has been removed it is safe to remove the configuration.
Removing the configuration is not mandatory and it can be reused on other clusters.

```sh
ibmcloud sat storage config rm --config <config-name>
```

## Troubleshooting, support, and logs

For full instructions, see the following documentation, referred to in the introductory paragraph.
**Troubleshooting**

## References

- IBM block storage CSI driver documentation: https://www.ibm.com/support/knowledgecenter/SSRQ8T