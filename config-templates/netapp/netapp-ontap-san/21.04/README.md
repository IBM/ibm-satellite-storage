# NetApp Trident - ONTAP SAN Driver

You can use the `netapp-ontap-san` Satellite storage template to deploy NetApp storage drivers which you can use dynamically provision and manage iSCSI LUNs in your ONTAP storage.

## Prerequisites

**Planning considerations for the Infrastructure Admin**
* Create a cluster that meets the requirements for ONTAP SAN. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v21.04/support/requirements.html). Verify that your backend ONTAP cluster is configured as a Trident backend.
   * You must have a dedicated Storage Virtual Machine (SVM) for Trident. Volumes and LUNs that are created by Trident are created in this SVM.
   * You must have one or more aggregates assigned to the SVM. 
   ### Example:
   ```sh
   netapp1::> vserver modify -vs <svm_name> -aggr-list <aggregate(s)_to_be_added>
   ```
   * You must have one or more dataLIFs for the SVM.
   * You must have iSCSI services enabled on the SVM.
   * You must set up a snapshot policy on the SVM.
* Share following details with Location Admin.


**Planning consideration for Location Admin**
* Make sure `netapp-trident` is deployed on the target cluster.
* Review the template parameters and retrieve the values from your NetApp cluster.

## NetApp Ontap-SAN Driver parameters & how to retrieve them

List the template parameters.
```
ibmcloud sat storage template get --name netapp-ontap-san --version 21.04
```

**NetApp Ontap-SAN Driver parameters**

| Parameter name | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `managementLIF` | Required | The IP address of the management LIF. Example: `10.0.0.1`. | N/A |
| `dataLIF` | Required | The IP address of data LIF. Example: `10.0.0.2`. | N/A | 
| `svm` | Required | The name of the storage virtual machine. Example: `svm-iscsi`. | N/A | 
| `username` | Required | The username to connect to the storage device. | N/A |
| `password` | Required | The password to connect to the storage device. | N/A |
| `limitVolumeSize` | Optional | Maximum volume size that can be requested and qtree parent volume size. Example: `50Gi`| `(not enforced by default)` |
| `limitAggregateUsage` | Optional | Limit provisioning of volumes if parent volume usage exceeds this value. For example, if a volume is requested that causes parent volume usage to exceed this value, the volume provisioning fails. Example: `80%`  | `(not enforced by default)` |


## Default storage classes

The following storage classes are installed when you assign your `netapp-ontap-san` configuration to your clusters to allow you to take advantage of ONTAP's QoS features:

| Storage class name | Type | File system | IOPs | Reclaim policy |
| --- | --- | --- | --- | --- |
| `ntap-block-gold` | Ontap-SAN | Block | user defined**\*** | Delete |
| `ntap-block-silver` | Ontap-SAN | Block | user defined**\*** | Delete |
| `ntap-block-bronze` | Ontap-SAN | Block | user defined**\*** | Delete | 
| `ntap-block-default` | Ontap-SAN | Block | n/a | Delete | 

**\*NOTE**: In order to use the **ntap-block-gold**, **ntap-block-silver** or **ntap-block-bronze** storage classes there must be QoS Policy groups named: **gold**, **silver**, and/or **bronze** on the storage controller. For information on creating and managing QoS Policy groups, please refer to the [ONTAP 9 Storage Management documentation](https://docs.netapp.com/ontap-9/index.jsp). If you choose not to use QoS policies, you must use the **ntap-block-default** storage class to create your PVCs.

## Creating the NetApp Ontap-SAN Driver storage configuration

Create a Satellite storage configuration that uses the `netapp-ontap-san` template.

**Example `sat storage config create` command**
Create a Satellite storage configuration by using the `netapp-ontap-san` template.
```
ibmcloud sat storage config create --name 'ontapsan-config' --template-name 'netapp-ontap-san' --template-version '21.04' -p 'managementLIF=10.0.0.1' -p 'dataLIF=10.0.0.2' -p 'svm=svm-san' -p 'username=admin' -p 'password=<admin password>'
```

## Creating the storage assignment
Assign your `netapp-ontap-san` storage configuration to your clusters.
**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name 'ontapsan-driver' --group <group name> --config 'ontapsan-config'
```

## Verifying your NetApp Ontap-SAN Driver storage configuration is assigned to your clusters

Verify that your `netapp-ontap-san` configuration is successfully assigned to your clusters. Run the following commands to verify that the driver pods and other Kubernetes resources are deployed.

```
oc -n trident get all  | grep 'trident-kubectl-san'
```
```
oc get sc | grep 'ntap-block'
```

**Example output**

```
$ oc -n trident get all  | grep 'trident-kubectl-san'
pod/trident-kubectl-san                 1/1     Running   0          48s
```
```
$ oc get sc | grep 'ntap-block'
ntap-block-bronze    csi.trident.netapp.io          Delete          Immediate              false                  2m36s
ntap-block-gold      csi.trident.netapp.io          Delete          Immediate              false                  2m37s
ntap-block-silver    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
ntap-block-default    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
```

## Troubleshooting

In case the PVC is not getting created using the `sat-netapp-block` storage classes
- Review the input parameters values for `managementLIF`, `dataLIF`, `svm`, `username`, `password` and other
- Review the `trident-kubectl-san` POD's log
  ```
  oc -n trident logs trident-kubectl-san
  ```
- Delete the assignment by running command `ibmcloud sat storage assignment rm --assignment <assignmnet name>`
- Delete the configuration by running command `ibmcloud sat storage config rm --config <config name>`
- Recreate the configuration, with proper parameter values, and recreate the assignment


## Reference

- https://netapp-trident.readthedocs.io/en/stable-v21.04/kubernetes/operations/tasks/backends/ontap/ontap-san/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v21.04/support/support.html
