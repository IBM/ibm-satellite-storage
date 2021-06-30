# NetApp Trident - ONTAP NAS Driver

You can use the `netapp-ontap-nas` Satellite storage template to deploy NetApp storage drives which you can use dynamically provision and mange file volumes in your ONTAP NAS.

## Prerequisites

**Planning consideration for Infra Admin**
* Create a cluster that meets the requirements for ONTAP NAS. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v20.07/support/requirements.html). Verify that your backend ONTAP cluster is configured as a Trident backend.
   * You must have a dedicated SVM for Trident. Volumes that are created by Trident are created in this SVM.
   * You must have one or more aggregates assigned to the SVM.
   ### Example:
   ```sh
   netapp1::> vserver modify -vs <svm_name> -aggr-list <aggregate(s)_to_be_added>
   ```
   * You must configure permissions on the default export policy or create your own custom export policy.
   * You must have one or more dataLIFs for the SVM. Depending on the protocol used (NFS/iSCSI), at least one dataLIF is required.
   * You must have NFS services enabled on the SVM.
   * You must set up a snapshot policy on the SVM.
* Share following details with Location Admin


**Planning consideration for Location Admin**
* Make sure `netapp-trident` is deployed on the target cluster
* Review the template parameters and retrieve the values from your NetApp cluster.

## NetApp Ontap-NAS Driver parameters & how to retrieve them

List the template parameters.
```
ibmcloud sat storage template get --name netapp-ontap-nas --version 20.7
```

**NetApp Ontap-NAS Driver parameters**

| Parameter name | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `managementLIF` | Required | The IP address of the management LIF. Example: `10.0.0.1`. | N/A |
| `dataLIF` | Required | The IP address of data LIF. Example: `10.0.0.2`. | N/A | 
| `svm` | Required | The name of the storage virtual machine. Example: `svm-nfs`. | N/A | 
| `export-policy` | Required | The NAS option for the NFS export policy. Example: `default`. | N/A |
| `username` | Required | The username to connect to the storage device. | N/A |
| `password` | Required | The password to connect to the storage device. | N/A |
| `limitVolumeSize` | Optional | Maximum volume size that can be requested and qtree parent volume size. | `50Gi` |
| `limitAggregateUsage` | Optional | Limit provisioning of volumes if parent volume usage exceeds this value. For example, if a volume is requested that causes parent volume usage to exceed this value, the volume provisioning fails.  | `80%` |
| `nfsMountOptions` | Optional | Specify the NFS mount version. Example: `nfsvers=4` | `nfsvers=4` |


## Default storage classes

The following storage classes are installed when you assign your `netapp-ontap-nas` configuration to your clusters.

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| `sat-netapp-file-gold` | Ontap-NAS | NFS | -- | -- | -- | Delete |
| `sat-netapp-file-silver` | Ontap-NAS | NFS | -- | -- | -- | Delete |
| `sat-netapp-file-bronze` | Ontap-NAS | NFS | -- | -- | -- | Delete | 


## Creating the NetApp Ontap-NAS Driver storage configuration

Create a Satellite storage configuration that uses the `netapp-ontap-nas` template.

**Example `sat storage config create` command**
Create a Satellite storage configuration by using the `netapp-ontap-nas` template.
```
ibmcloud sat storage config create --name 'ontapnas-config' --template-name 'netapp-ontap-nas' --template-version '20.07' -p 'managementLIF=10.0.0.1' -p 'dataLIF=10.0.0.2' -p 'svm=svm-nas' -p 'username=admin' -p 'password=<admin password>' -p 'exportPolicy=nfsexport'
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
oc get sc | grep 'netapp-file'
```

**Example output**

```
$ oc -n trident get all  | grep 'trident-kubectl-nas'
pod/trident-kubectl-nas                 1/1     Running   0          48s
```
```
$ oc get sc | grep 'netapp-file'
sat-netapp-file-bronze    csi.trident.netapp.io          Delete          Immediate              false                  2m36s
sat-netapp-file-gold      csi.trident.netapp.io          Delete          Immediate              false                  2m37s
sat-netapp-file-silver    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
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

- https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-nas/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html
