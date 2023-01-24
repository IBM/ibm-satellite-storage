# NetApp Trident - ONTAP NAS Driver

You can use the `netapp-ontap-nas` Satellite storage template to deploy NetApp storage drivers which you can use to dynamically provision and manage NFS volumes in your ONTAP storage.

## Prerequisites

**Planning consideration for Infra Admin**
* Create a cluster that meets the requirements for ONTAP NAS. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v21.04/support/requirements.html). 
* Verify that your backend ONTAP cluster is configured as a Trident backend.
* You must have a dedicated Storage Virtual Machine (SVM) for Trident. Volumes that are created by Trident are created in this SVM.
* You must have one or more aggregates assigned to the SVM. You can add aggregates with the `netapp1::> vserver modify -vs <svm_name> -aggr-list <aggregate(s)_to_be_added>` command.
* You must configure permissions on the default export policy or create your own custom export policy.
* You must have one or more dataLIFs for the SVM.
* You must have NFS services enabled on the SVM.
* You must set up a snapshot policy on the SVM.

**Planning considerations for the Satellite Location Admin**
* Make sure `netapp-trident` is deployed on the target cluster
* Review the template parameters and retrieve the values from your NetApp cluster.

## NetApp Ontap-NAS Driver parameters & how to retrieve them

List the template parameters.
```
ibmcloud sat storage template get --name netapp-ontap-nas --version 22.10
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
| `limitVolumeSize` | Optional | Maximum volume size that can be requested per PVC. Example: `500Gi` | `default is 50Gi` |
| `limitAggregateUsage` | Optional | Limit provisioning of volumes if parent volume usage exceeds this value. For example, if a volume is requested that causes parent volume usage to exceed this value, the volume provisioning fails. Example: `70%` | `default is 80%` |
| `nfsMountOptions` | Optional | Specify the NFS mount version. Example: `nfsvers=4` | `""` |


## Default storage classes

You can use the `sat-netapp` storage classes to take advantage of ONTAP's QoS features. The following storage classes are installed when you assign your `netapp-ontap-nas` configuration to your clusters. Review the following notes before deploying an app that uses one of the `sat-netapp` storage classes.

**Notes**: 
* By default, the `sat-netapp-file-gold` storage class has no QoS limits (unlimited IOPS). 
* In order to use the `sat-netapp-file-silver` and `sat-netapp-file-bronze` storage classes, you must create corresponding `silver` and `bronze` QoS policy groups on the storage controller and define the QoS limits. To create a policy group on the storage system, login to the system CLI and running the `netapp1::> qos policy-group create -policy-group <policy_group_name> -vserver <svm_name> [-min-throughput <min_IOPS>] -max-throughput <max_IOPS>` command.
* ***min-throughput*** is only supported on all-flash systems. For information on creating and managing QoS Policy groups, please refer to the [ONTAP 9 Storage Management documentation](https://docs.netapp.com/ontap-9/index.jsp).
* In order to use an ***encrypted*** storage class, NetApp Volume Encryption (NVE) must be enabled on your storage system using either the NetApp ONTAP onboard key manager or a supported (off-box) third-party key manager, such as IBM's TKLM key manager. To enable the onboard key manager, run the `netapp1::> security key-manager onboard enable` command. For more information on configuring encryption, please refer to the [ONTAP 9 Security and Data Encryption documentation](https://docs.netapp.com/ontap-9/topic/com.netapp.nav.aac/home.html?cp=14).

| Storage class name | Type | File system | IOPs | Encryption | Reclaim policy |
| --- | --- | --- | --- | --- | --- |
| `sat-netapp-file-gold` | Ontap-NAS | NFS | no QoS limits | Encryption disabled. | Delete |
| `sat-netapp-file-gold-encrypted` | Ontap-NAS | NFS | no QoS limits | Encryption enabled. | Delete |
| `sat-netapp-file-silver` | Ontap-NAS | NFS | User defined QoS limit. | Encryption disabled. | Delete |
| `sat-netapp-file-silver-encrypted` | Ontap-NAS | NFS | User defined QoS limit. | Encryption enabled. | Delete |
| `sat-netapp-file-bronze` | Ontap-NAS | NFS | User-defined QoS limit. | Encryption disabled. | Delete |
| `sat-netapp-file-bronze-encrypted` | Ontap-NAS | NFS | User-defined QoS limit.| Encryption enabled. | Delete |



## Creating the NetApp Ontap-NAS Driver storage configuration

Create a Satellite storage configuration that uses the `netapp-ontap-nas` template.

**Example `sat storage config create` command**
Create a Satellite storage configuration by using the `netapp-ontap-nas` template.

```
ibmcloud sat storage config create --name 'ontapnas-config' --location <location id> --template-name 'netapp-ontap-nas' --template-version '22.10' -p 'managementLIF=10.0.0.1' -p 'dataLIF=10.0.0.2' -p 'svm=svm-nas' -p 'username=admin' -p 'password=<admin password>' -p 'exportPolicy=nfsexport'
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

If PVCs are not getting created using the `sat-netapp-file` storage classes, complete the following troubleshooting steps.
- Review your input parameters and verify the `managementLIF`, `dataLIF`, `svm`, `username`, `password`.
- Gather the logs from  `trident-kubectl-nas` pod.
  ```
  oc -n trident logs trident-kubectl-nas
  ```
  
To recreate your configuration, complete the following steps.
1. Delete the assignment by running `ibmcloud sat storage assignment rm --assignment <assignmnet name>`.
2. Delete the configuration by running command `ibmcloud sat storage config rm --config <config name>`.
3. Recreate the configuration, with proper parameter values, and recreate the assignment.


## Reference

- NetApp docs: https://netapp-trident.readthedocs.io/en/stable-v21.04/kubernetes/operations/tasks/backends/ontap/ontap-nas/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v21.04/support/support.html
