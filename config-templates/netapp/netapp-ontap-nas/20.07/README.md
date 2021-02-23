# # NetApp Trident - ONTAP NAS Driver

The ontap-nas driver provides the capability to dynamically provision and mange the lifecycle of the file volumes on Ontap.

## Prerequisites

**Planning consideration for Infra Admin**
* Setup Ontap NAS Cluster
* Share following details with Location Admin
   - managementLIF
   - dataLIF
   - svm
   - username
   - password
* The backend ONTAP cluster has been configured to be used as a Trident backend. Namely:
   * A SVM must be dedicated for Trident. Volumes created by Trident will be placed in this SVM.
   * One or more aggregates must be assigned to the SVM.
   * One or more dataLIFs must be created for the SVM. Depending on the protocol used (NFS/iSCSI), at least one dataLIF is required.
   * NFS services must be started on the SVM.
   * Ensure the SVM has snapshot policies set up to be used by Trident. It is recommended a default snapshot policy is created on the SVM being used.

## NetaApp Ontap-NAS Driver parameters & how to retrieve them

Retrieve all parameters required by this template
```
ibmcloud sat storage template get --name netapp-ontap-nas --version 20.7
```

**NetaApp Ontap-NAS Driver parameters**

| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `namespace` | Optional | Namespace for driver deployment | `trident-nas` |
| `managementLIF` | Required | IP address of ONTAP management LIF | `N/A` |
| `dataLIF` | Required | IP address of protocol LIF | `N/A` |
| `svm` | Required | Storage virtual machine to use | `N/A` |
| `username` | Required | Username to connect to the storage device | `N/A` |
| `password` | Required | Password to connect to the storage device | `N/A` |
| `exportPolicy` | Optional | NAS option for the NFS export policy to use | `default` |
| `limitVolumeSize` | Optional | Maximum requestable volume size and qtree parent volume size | `50Gi` |
| `limitAggregateUsage` | Optional | Fail provisioning if usage is above this percentage | `80%` |
| `nfsMountOptions` | Optional | Fine grained control of NFS mount options; defaults to -o nfsvers=4 | `nfsvers=4` |


## Default storage classes

Provide a table of the Satellite storage classes that are installed when your configuration is assigned to a cluster. Refer to the following example table. Add or remove storage class details based on the storage type.

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| `sat-netapp-file-gold` | Ontap-NAS | NFS | -- | -- | -- | Delete |
| `sat-netapp-file-silver` | Ontap-NAS | NFS | -- | -- | -- | Delete |
| `sat-netapp-file-bronze` | Ontap-NAS | NFS | -- | -- | -- | Delete | 


## Creating the NetaApp Ontap-NAS Driver storage configuration

Provide example steps, including an example `config create` command for creating a Satellite storage configuration that uses your template.

**Example `sat storage config create` command**

```
ibmcloud sat storage config create --name ontapnas --template-name netapp-ontap-nas --template-version 20.07 -p "managementLIF=10.0.0.1" -p "dataLIF=10.0.0.2" -p "svm=svm-nas" -p "username=admin" -p "password=admin-passW@rd" -p "exportPolicy=nfsexport"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```
ibmcloud sat storage assignment create --name nasdriver --group filestore --config ontapnas
```

## Verifying your NetaApp Ontap-NAS Driver storage configuration is assigned to your clusters

Provide steps to retrieve any driver pods, storage classes, or any other resources that are deployed with your storage configuration.

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

- https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-nas/index.html
- Support: https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html
