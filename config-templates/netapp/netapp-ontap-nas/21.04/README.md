# NetApp Trident - ONTAP NAS Driver

You can use the `netapp-ontap-nas` Satellite storage template to deploy NetApp storage drivers which you can use to dynamically provision and manage NFS volumes in your ONTAP storage.

## Prerequisites

**Planning consideration for Infra Admin**
* Create a cluster that meets the requirements for ONTAP NAS. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v21.04/support/requirements.html). Verify that your backend ONTAP cluster is configured as a Trident backend.
   * You must have a dedicated Storage Virtual Machine (SVM) for Trident. Volumes that are created by Trident are created in this SVM.
   * You must have one or more aggregates assigned to the SVM.
   * You must have one or more dataLIFs for the SVM.
   * You must have NFS services enabled on the SVM.
   * You must set up a snapshot policy on the SVM.
* Share following details with Location Admin


**Planning considerations for the Satellite Location Admin**
* Make sure `netapp-trident` is deployed on the target cluster
* Review the template parameters and retrieve the values from your NetApp cluster.

## NetApp Ontap-NAS Driver parameters & how to retrieve them

List the template parameters.
```
ibmcloud sat storage template get --name netapp-ontap-nas --version 21.04
```

**NetApp Ontap-NAS Driver parameters**

| Parameter name | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `managementLIF` | Required | The IP address of the management LIF. Example: `10.0.0.1`. | N/A |
| `dataLIF` | Required | The IP address of data LIF. Example: `10.0.0.2`. | N/A | 
| `svm` | Required | The name of the storage virtual machine. Example: `svm-nfs`. | N/A | 
| `export-policy` | Optional | Provide the name of an NFS export policy to use. Example: `remote-workers`. | `default` |
| `username` | Required | The username to connect to the storage device. | N/A |
| `password` | Required | The password to connect to the storage device. | N/A |
| `limitVolumeSize` | Optional | Maximum volume size that can be requested and qtree parent volume size. Example: `50Gi` | `"" (not enforced by default)` |
| `limitAggregateUsage` | Optional | Limit provisioning of volumes if parent volume usage exceeds this value. For example, if a volume is requested that causes parent volume usage to exceed this value, the volume provisioning fails.  | `80%` |
| `nfsMountOptions` | Optional | Specify the NFS mount version. Example: `nfsvers=4` | `""` |


## Default storage classes

The following storage classes are installed when you assign your `netapp-ontap-nas` configuration to your clusters to allow you to take advantage of ONTAP's QoS features:

| Storage class name | Type | File system | IOPs | Reclaim policy |
| --- | --- | --- | --- | --- |
| `ntap-file-gold` | Ontap-NAS | NFS | user defined**\*** | Delete |
| `ntap-file-silver` | Ontap-NAS | NFS | user defined**\*** | Delete |
| `ntap-file-bronze` | Ontap-NAS | NFS | user defined**\*** | Delete | 
| `ntap-file-default` | Ontap-NAS | NFS | n/a | Delete | 

**\*NOTE**: In order to use the **ntap-file-gold**, **ntap-file-silver** or **ntap-file-bronze** storage classes, the must be QoS Policy groups named **gold**, **silver**,and/or **bronze** on the storage controller.
For information on creating and managing QoS Policy groups, please refer to the [ONTAP 9 Storage Management documentation](https://docs.netapp.com/ontap-9/index.jsp).  If you choose not to use QoS policies, you must use the **ntap-file-default** storage class to create your PVCs.


## Creating the NetApp Ontap-NAS Driver storage configuration

Create a Satellite storage configuration that uses the `netapp-ontap-nas` template.

**Example `sat storage config create` command**
Create a Satellite storage configuration by using the `netapp-ontap-nas` template.
```
ibmcloud sat storage config create --name 'ontapnas-config' --template-name 'netapp-ontap-nas' --template-version '21.04' -p 'managementLIF=10.0.0.1' -p 'dataLIF=10.0.0.2' -p 'svm=svm-nas' -p 'username=admin' -p 'password=<admin password>' -p 'exportPolicy=nfsexport'
```

## Creating the storage assignment
Assign your `netapp-ontap-nas` storage configuration to your clusters.
**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name 'ontapnas-driver' --group <group name> --config 'ontapnas-config'
```

## Verifying your NetApp Ontap-NAS Driver storage configuration is assigned to your clusters

Verify that your `netapp-ontap-nas` configuration is successfully assigned to your clusters. Run the following commands to verify that the driver pods and other Kubernetes resources are deployed.

```
oc -n trident get all  | grep 'trident-kubectl-nas'
```
```
oc get sc | grep 'ntap-file'
```

**Example output**

```
$ oc -n trident get all  | grep 'trident-kubectl-nas'
pod/trident-kubectl-nas                 1/1     Running   0          48s
```
```
$ oc get sc | grep 'ntap-file'
ntap-file-bronze    csi.trident.netapp.io          Delete          Immediate              false                  2m36s
ntap-file-gold      csi.trident.netapp.io          Delete          Immediate              false                  2m37s
ntap-file-silver    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
ntap-file-default    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
```

## Troubleshooting

In case the PVC is not getting created using the `sat-netapp-file` storage classes
- Review the input parameters values for `managementLIF`, `dataLIF`, `svm`, `username`, `password` and other
- Review the `trident-kubectl-nas` POD's log
  ```
  oc -n trident logs trident-kubectl-nas
  ```
- Delete the assignment by running command `ibmcloud sat storage assignment rm --assignment <assignmnet name>`
- Delete the configuration by running command `ibmcloud sat storage config rm --config <config name>`
- Recreate the configuration, with proper parameter values, and recreate the assignment


## Reference

- https://netapp-trident.readthedocs.io/en/stable-v21.04/kubernetes/operations/tasks/backends/ontap/ontap-nas/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v21.04/support/support.html
