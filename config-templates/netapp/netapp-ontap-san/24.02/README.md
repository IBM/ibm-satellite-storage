# NetApp Trident - ONTAP SAN Driver

You can use the `netapp-ontap-san` Satellite storage template to deploy NetApp storage drivers which you can use dynamically provision and manage iSCSI LUNs in your ONTAP storage.

## Prerequisites

**Planning considerations for the Infrastructure Admin**
* Create a cluster that meets the requirements for ONTAP SAN. For more information, see the [NetApp documentation](https://docs.netapp.com/us-en/trident/trident-get-started/requirements.html).
* Verify that your backend ONTAP cluster is configured as a Trident backend.
* You must have a dedicated Storage Virtual Machine (SVM) for Trident. Volumes and LUNs that are created by Trident are created in this SVM.
* You must have one or more aggregates assigned to the SVM. You can add aggregates by running the `netapp1::> vserver modify -vs <svm_name> -aggr-list <aggregate(s)_to_be_added>` command.
* You must have one or more `dataLIFs` for the SVM.
* You must have iSCSI services enabled on the SVM.
* You must set up a snapshot policy on the SVM.

**Planning consideration for Location Admin**
* Make sure `netapp-trident` is deployed on the target cluster.
* Review the template parameters and retrieve the values from your NetApp cluster.

## NetApp Ontap-SAN Driver parameters & how to retrieve them

List the template parameters.
```
ibmcloud sat storage template get --name netapp-ontap-san --version 24.02
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

You can use the `sat-netapp` storage classes to take advantage of ONTAP's QoS features. The following storage classes are installed when you assign your `netapp-ontap-nas` configuration. Review the following notes before deploying an app that uses one of the `sat-netapp` storage classes.

**Notes**: 
* By default, the `sat-netapp-file-gold` storage class has no QoS limits (unlimited IOPS). 
* In order to use the `sat-netapp-file-silver` and `sat-netapp-file-bronze` storage classes, you must create corresponding `silver` and `bronze` QoS policy groups on the storage controller and define the QoS limits. To create a policy group on the storage system, login to the system CLI and running the `netapp1::> qos policy-group create -policy-group <policy_group_name> -vserver <svm_name> [-min-throughput <min_IOPS>] -max-throughput <max_IOPS>` command.
* **min-throughput** is only supported on all-flash systems. For information on creating and managing QoS Policy groups, please refer to the [ONTAP 9 Storage Management documentation](https://docs.netapp.com/ontap-9/index.jsp).
* In order to use an **encrypted** storage class, NetApp Volume Encryption (NVE) must be enabled on your storage system using either the NetApp ONTAP onboard key manager or a supported (off-box) third-party key manager, such as IBM's TKLM key manager. To enable the onboard key manager, run the `netapp1::> security key-manager onboard enable` command. For more information on configuring encryption, please refer to the [ONTAP 9 Security and Data Encryption documentation](https://docs.netapp.com/ontap-9/topic/com.netapp.nav.aac/home.html?cp=14).

| Storage class name | Type | File system | IOPs | Encryption |Reclaim policy |
| --- | --- | --- | --- | --- | --- |
| `sat-netapp-block-gold` | Ontap-SAN | ext4 | no QoS limits. | Encryption disabled. | Delete |
| `sat-netapp-block-gold-encrypted` | Ontap-SAN | ext4 | no QoS limits. | Encryption enabled. | Delete |
| `sat-netapp-block-silver` | Ontap-SAN | ext4 | User-defined QoS limit. | Encryption disabled. | Delete |
| `sat-netapp-block-silver-encrypted` | Ontap-SAN | ext4 | User-defined QoS limit. | Encryption enabled. | Delete |
| `sat-netapp-block-bronze` | Ontap-SAN | ext4 | User defined QoS limit. | Encryption disabled. | Delete |
| `sat-netapp-block-bronze-encrypted` | Ontap-SAN | ext4 | User-defined QoS limit. | Encryption enabled. | Delete |



## Creating the NetApp Ontap-SAN Driver storage configuration

Create a Satellite storage configuration that uses the `netapp-ontap-san` template.

**Example `sat storage config create` command**
Create a Satellite storage configuration by using the `netapp-ontap-san` template.
```
ibmcloud sat storage config create --name 'ontapsan-config' --location <location id> --template-name 'netapp-ontap-san' --template-version '22.10' -p 'managementLIF=10.0.0.1' -p 'dataLIF=10.0.0.2' -p 'svm=svm-san' -p 'username=admin' -p 'password=<admin password>' // pragma: allowlist secret
```

## Creating the storage assignment
Assign your `netapp-ontap-san` storage configuration to your clusters.


**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name 'ontapsan-driver' --group <group name> --config 'ontapsan-config'
```

## Verifying your NetApp Ontap-SAN Driver storage configuration is assigned to your clusters

Verify that your `netapp-ontap-san` configuration is successfully assigned to your clusters. Run the following commands to verify that the driver pods and other Kubernetes resources are deployed.

```sh
oc -n trident get all  | grep 'trident-kubectl-san'
```
```sh
oc get sc | grep 'ntap-block'
```

**Example output**

```sh
$ oc -n trident get all  | grep 'trident-kubectl-san'
pod/trident-kubectl-san                 1/1     Running   0          48s
```
```sh
$ oc get sc | grep 'ntap-block'
ntap-block-bronze    csi.trident.netapp.io          Delete          Immediate              false                  2m36s
ntap-block-gold      csi.trident.netapp.io          Delete          Immediate              false                  2m37s
ntap-block-silver    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
ntap-block-default    csi.trident.netapp.io          Delete          Immediate              false                  2m37s
```

## Troubleshooting

If PVC creation fails, complete the following troubleshooting steps.
1. Rreview the input parameters values and verify the `managementLIF`, `dataLIF`, `svm`, `username`, `password`.
Gete the logs of the `trident-kubectl-san` POD's log.
  ```sh
  oc -n trident logs trident-kubectl-san
  ```
  
To recreate your configuration, complete the following steps.
1. Delete the assignment by running command `ibmcloud sat storage assignment rm --assignment <assignmnet name>`
2. Delete the configuration by running command `ibmcloud sat storage config rm --config <config name>`.
3. Recreate the configuration with the correct parameters and recreate the assignment.

If app-pod is struct in `ContainerCreating` state and error in the events of the app-pod are related to `multipathd` do following:

**ERROR:** 
```
  Warning  FailedMount             108s (x14 over 14m)  kubelet                  MountVolume.MountDevice failed for volume "pvc-7f06dbb7-41ed-4db5-95d8-51a48102293c" : rpc error: code = Internal desc = failed to stage volume: multipathd is not running
```

**Solution:**

Run the following commands to enable and start `multipathd` service in worker nodes: 
```
# mpathconf --enable
# systemctl start multipathd.service
```

**ERROR:** 
```
  Warning  FailedMount             11s (x6 over 27s)  kubelet                  MountVolume.MountDevice failed for volume "pvc-7f06dbb7-41ed-4db5-95d8-51a48102293c" : rpc error: code = Internal desc = failed to stage volume: multipathd: unsupported find_multipaths: yes value; please set the value to no in /etc/multipath.conf file
```

**Solution:**

Run the following commands in worker nodes: 
1. `mpathconf --enable --find_multipaths n`
2. Check the status of `multipathd` service:
     - If it is `Active`, then restart the service: `systemctl restart multipathd.service`
     - Otherwise, start the service: `systemctl start multipathd.service`

## Reference

- NetApp docs: https://docs.netapp.com/us-en/trident/trident-use/ontap-san-prep.html
- Support: https://docs.netapp.com/us-en/trident/get-help.html#astra-trident-support-lifecycle
