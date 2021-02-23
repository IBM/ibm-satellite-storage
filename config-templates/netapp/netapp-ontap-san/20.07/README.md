# NetaApp Ontap-SAN

You can use the `netapp-ontap-san` driver to dynamically provision and mange ONTAP-SAN block volumes.

## Prerequisites

* Create a cluster that meets the requirements for ONTAP SAN. For more information, see the [NetApp documentation](https://netapp-trident.readthedocs.io/en/stable-v20.07/support/requirements.html). Verify that your backend ONTAP cluster is configured as a Trident backend.
* Share following details with Location Admin
   - managementLIF
   - dataLIF
   - svm
   - username
   - password
* The backend ONTAP cluster has been configured to be used as a Trident backend. Namely:
   * A SVM must be dedicated for Trident. Volumes created by Trident will be placed in this SVM.
   * One or more aggregates must be assigned to the SVM.
   * One or more dataLIFs must be created for the SVM. Depending on the protocol used (iSCSI/NFS), at least one dataLIF is required.
   * Ensure the SVM has snapshot policies set up to be used by Trident. It is recommended a default snapshot policy is created on the SVM being used.

## NetaApp Ontap-SAN Driver parameters & how to retrieve them

Retrieve all parameters required by this template
```
ibmcloud sat storage template get --name netapp-ontap-san --version 20.7
```

**NetaApp Ontap-SAN Driver parameters**

| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `namespace` | Optional | Namespace for driver deployment | `trident-nas` |
| `managementLIF` | Required | IP address of ONTAP management LIF | `N/A` |
| `dataLIF` | Required | IP address of protocol LIF | `N/A` |
| `svm` | Required | Storage virtual machine to use | `N/A` |
| `username` | Required | Username to connect to the storage device | `N/A` |
| `password` | Required | Password to connect to the storage device | `N/A` |
| `limitVolumeSize` | Optional | The maximum volume size that can be requested and the qtree parent volume size. | `50Gi` |
| `limitAggregateUsage` | Optional | Limit the provisioning of volumes if the parent volume usage exceeds this value. For example, if a volume is requested that causes the parent volume usage to exceed this value, the volume provisioning fails.  | `80%` |

## Default storage classes

The following storage classes are deployed when you assign your NetApp ONTAP-SAN configuration to your clusters.

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| `sat-netapp-block-gold` | Ontap-SAN | Block | -- | -- | -- | Delete |
| `sat-netapp-block-silver` | Ontap-SAN | Block | -- | -- | -- | Delete |
| `sat-netapp-block-bronze` | Ontap-SAN | Block | -- | -- | -- | Delete | 


## Creating the NetaApp Ontap-SAN Driver storage configuration

Provide example steps, including an example `config create` command for creating a Satellite storage configuration that uses your template.

**Example `sat storage config create` command**

```
ibmcloud sat storage config create --name ontapsan --template-name netapp-ontap-san --template-version 20.07 -p "managementLIF=10.0.0.1" -p "dataLIF=10.0.0.2" -p "svm=svm-nas" -p "username=admin" -p "password=admin-passW@rd"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name sandriver --group blockstore --config ontapsan
```

## Verifying your NetApp Ontap-SAN configuration is assigned to your clusters.


```
oc get pods 
```

```
oc get sc | grep sat
```


**Example output**

Provide an example output screen of running driver pods and storage classes that are deployed by your configuration.


## Troubleshooting

Provide troubleshooting steps for any known issues.


## Reference

- https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-san/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html
