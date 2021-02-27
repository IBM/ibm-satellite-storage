# NetApp Ontap-SAN

You can use the `netapp-ontap-san` driver to dynamically provision and mange ONTAP-SAN block volumes.

## Prerequisites

**Planning consideration for Infra Admin**
* Create a cluster that meets the requirements for ONTAP SAN. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v20.07/support/requirements.html). Verify that your backend ONTAP cluster is configured as a Trident backend.
Review the template parameters and retrieve the values from your NetApp cluster.
* Create a cluster that meets the requirements for ONTAP NAS. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v20.07/support/requirements.html). Verify that your backend ONTAP cluster is configured as a Trident backend.
   * You must have a dedicated SVM for Trident. Volumes that are created by Trident are created in this SVM.
   * You must have one or more aggregates assigned to the SVM.
   * You must have one or more dataLIFs for the SVM. Depending on the protocol used (NFS/iSCSI), at least one dataLIF is required.
   * You must have NFS services enabled on the SVM.
   * You must set up a snapshot policy on the SVM.

**Planning consideration for Location Admin**
* Ensure the NetApp Trident Operator is installed on the target cluster.
* Review the template parameters and retrieve the values from your NetApp cluster.


## NetApp Ontap-SAN Driver parameters & how to retrieve them

List the template parameters.
```
ibmcloud sat storage template get --name netapp-ontap-san --version 20.7
```

**NetApp Ontap-SAN Driver parameters**

| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `managementLIF` | Required | The IP address of the management LIF. Example: `10.0.0.1`. | N/A |
| `dataLIF` | Required | The IP address of data LIF. Example: `10.0.0.2`. | N/A | 
| `svm` | Required | The name of the storage virtual machine. Example: `svm`. | N/A | 
| `username` | Required | The username to connect to the storage device. | N/A |
| `password` | Required | The password to connect to the storage device. | N/A |
| `limitVolumeSize` | Optional | The maximum volume size that can be requested and the qtree parent volume size. | `50Gi` |
| `limitAggregateUsage` | Optional | Limit the provisioning of volumes if the parent volume usage exceeds this value. For example, if a volume is requested that causes the parent volume usage to exceed this value, the volume provisioning fails.  | `80%` |

## Default storage classes

The following storage classes are deployed when you assign your NetApp ONTAP-SAN configuration to your clusters.

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| `sat-netapp-block-gold` | Ontap-SAN | Block | -- | -- | -- | Delete |
| `sat-netapp-block-silver` | Ontap-SAN | Block | -- | -- | -- | Delete |
| `sat-netapp-block-bronze` | Ontap-SAN | Block | -- | -- | -- | Delete | 


## Creating the NetApp Ontap-SAN Driver storage configuration

Create a Satellite storage configuration that uses the `netapp-ontap-san` template.

**Example `sat storage config create` command**

```
ibmcloud sat storage config create --name 'ontapsan-config' --template-name 'netapp-ontap-san' --template-version '20.07' -p 'managementLIF=10.0.0.1' -p 'dataLIF=10.0.0.2' -p 'svm=svm-nas' -p 'username=admin' -p 'password=<admin password>'
```

## Creating the storage assignment

**Example `sat storage assignment create` command**
Assign your storage configuration to your clusters.
```
ibmcloud sat storage assignment create --name 'ontapsan-driver' --group <group name> --config 'ontapsan-config'
```

## Verifying your NetApp Ontap-SAN configuration is assigned to your clusters.

```
oc -n trident get all | grep trident-kubectl-san
```
```
oc get sc | grep 'netapp-block'
```

**Example output**

```
$ oc -n trident get all | grep trident-kubectl-san
pod/trident-kubectl-san                 1/1     Running   0          37s
```
```
$ oc get sc | grep 'netapp-block'
sat-netapp-block-bronze   csi.trident.netapp.io          Delete          Immediate              false                  70s
sat-netapp-block-gold     csi.trident.netapp.io          Delete          Immediate              false                  71s
sat-netapp-block-silver   csi.trident.netapp.io          Delete          Immediate              false                  70s
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

- [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-san/index.html)
- Support: https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html
