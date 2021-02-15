# Red Hat OpenShift Container Storage

Red Hat OpenShift Container Storage is a software-defined storage that is optimised for container environments. It runs as an operator on OpenShift Container Platform to provide highly integrated and simplified persistent storage management for containers.

The user has to provide the input values to the custom resource OcsCluster while creating the satellite configuration to deploy OCS

## How to use the local template to deploy OCS on a Satellite Cluster having local storage?

### Prerequisites

1) In order to deploy OCS, at least one of these local storage options are required:
    - Raw devices (no partitions or formatted filesystems)
    - Raw partitions (no formatted filesystem)
    - PVs available from a storage class in block mode
2) The cluster needs to have a minimum of 3 nodes
3) The OCP version should be compatible with the OCS version you're trying to install
4) The nodes used for the cluster should have a configuration of minimum 16CPUs and 64GB RAM
5) Create the COS secret by following the steps given below :

a) Create the `openshift-storage` namespace using the yaml given below

```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
  name: openshift-storage
```

b) Create an IBM COS Service Instance
```
$ ibmcloud resource service-instance-create noobaa-stor cloud-object-storage standard global
```

c) Create access keys
```
$ ibmcloud resource service-key-create cos-cred-rw Writer --instance-name noobaa-stor --parameters '{ "HMAC": true}'
```

d) Create the Kubernetes secret under the `openshift-storage` namespace
```
$ kubectl -n 'openshift-storage' create secret generic 'ibm-cloud-cos-creds' --type=Opaque --from-literal=IBM_COS_ACCESS_KEY_ID=<access_key_id> --from-literal=IBM_COS_SECRET_ACCESS_KEY=<secret_access_key>
```

6) Note : for the `osd-device-path` and `mon-device-path` parameters, we need to find the disk by ID of the disks we want to use.

To find the disk by id of the disks :
a) Logon to each worker node that will be used for OCS using `oc debug node/<nodename>`, run `chroot /host` followed by `lsblk` to find available disks.
```
oc debug node/ip-10-0-135-71.us-east-2.compute.internal
Starting pod/ip-10-0-135-71us-east-2computeinternal-debug ...
To use host binaries, run `chroot /host`
Pod IP: 10.0.135.71
If you don't see a command prompt, try pressing enter.
sh-4.2# chroot /host
sh-4.4# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   931G  0 disk
|-sda1   8:1    0   256M  0 part /boot
|-sda2   8:2    0     1G  0 part
`-sda3   8:3    0 929.8G  0 part /
sdb      8:16   0 744.7G  0 disk
`-sdb1   8:17   0 744.7G  0 part /disk1
sdc      8:32   0 744.7G  0 disk
|-sdc1   8:33   0  18.6G  0 part
`-sdc2   8:34   0 260.8G  0 part
```

Note : Available disks that can be used for OCS are unmounted disks and in case of partitioned disks, we need to use the path of the partition for finding disk-by-id

b)After you know which local disks are available, in this case sdc1, and sdc2, you can now find the disk by-id, a unique name depending on the hardware serial number for each disk.

```
sh-4.2# ls -l /dev/disk/by-id/
total 0
lrwxrwxrwx. 1 root root  9 Feb  9 04:15 scsi-3600605b00d87b43027b3bbb603150cc6 -> ../../sda
lrwxrwxrwx. 1 root root 10 Feb  9 04:15 scsi-3600605b00d87b43027b3bbb603150cc6-part1 -> ../../sda1
lrwxrwxrwx. 1 root root 10 Feb  9 04:15 scsi-3600605b00d87b43027b3bbb603150cc6-part2 -> ../../sda2
lrwxrwxrwx. 1 root root 10 Feb  9 04:15 scsi-3600605b00d87b43027b3bbb603150cc6-part3 -> ../../sda3
lrwxrwxrwx. 1 root root  9 Feb  9 04:15 scsi-3600605b00d87b43027b3bbf306bc28a7 -> ../../sdb
lrwxrwxrwx. 1 root root 10 Feb  9 04:15 scsi-3600605b00d87b43027b3bbf306bc28a7-part1 -> ../../sdb1
lrwxrwxrwx. 1 root root  9 Feb  9 04:17 scsi-3600605b00d87b43027b3bc310a64c6c9 -> ../../sdc
lrwxrwxrwx. 1 root root 10 Feb 11 03:14 scsi-3600605b00d87b43027b3bc310a64c6c9-part1 -> ../../sdc1
lrwxrwxrwx. 1 root root 10 Feb 11 03:15 scsi-3600605b00d87b43027b3bc310a64c6c9-part2 -> ../../sdc2

```
c) From above, we can see that the disk by ids are :
   `scsi-3600605b00d87b43027b3bc310a64c6c9-part1`
   `scsi-3600605b00d87b43027b3bc310a64c6c9-part2`

###Red hat Openshift Container Storage :  Parameters & how to retrieve them

#### OCS local: Parameters

**Description of the template parameters :**

| Parameter | Required? | Description | Default value if not provided | Datatype |
| --- | --- | --- | --- | --- |
| `ocs-cluster-name` | Required | Enter the name of your OcsCluster custom resource. | N/A | string |
| `mon-device-path` | Required | Enter the `disk-by-id` paths to the devices that you want to use for the MON pods. Example: `/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part1`. | N/A | csv |
| `osd-device-path` | Required | Enter the `disk-by-id` paths to the devices that you want to use for the OSD pods. Example: `/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part2`. | N/A | csv |
| `num-of-osd` | Optional | Enter the number of OSDs. OCS will create 3x number of OSDs for the value specified. Initial storage capacity is the same as your disk size specified at `osd-device-path`. When you want to increase your storage capacity, you have to increase `num-of-osd` by the number of disks you add (taking into consideration the replication factor, which is `3` by default) | 1 | integer |
|worker-nodes | Optional | Workers which need to be a part of OCS (Minimum 3). If not specified, all workers will be considered for OCS | N/A | csv |
| `billing-type` | Optional | Enter the billing option that you want to use. You can enter either `hourly` or `monthly`. | `hourly` | string |
| `ocs-upgrade` | Optional | Set to `true` if you want to upgrade the major version of OCS. | false | boolean |

**Default storage classes**

| **Storage class name** | **Type** | **Provisioner** | **Reclaim policy**|
|------------------------|-----------------|--------|----------|
| sat-ocs-cephrbd-gold | Block | openshift-storage.rbd.csi.ceph.com | Delete |
| sat-ocs-cephfs-gold | File | openshift-storage.cephfs.csi.ceph.com | Delete |
| sat-ocs-cephrgw-gold | Object | openshift-storage.ceph.rook.io/bucket |Delete |
| sat-ocs-noobaa-gold | Object bucket | openshift-storage.noobaa.io/obc | Delete |

#### Creating the OCS storage configuration

##### Detailed steps

1. Login into the Cluster using oc CLI or IBM Cloud CLI
2. Verify that all the worker nodes are healthy.

```
$oc get nodes
NAME            STATUS   ROLES           AGE   VERSION
169.48.170.83   Ready    master,worker   28h   v1.19.0+3b01205
169.48.170.88   Ready    master,worker   28h   v1.19.0+3b01205
169.48.170.90   Ready    master,worker   28h   v1.19.0+3b01205
```

3. Create Cluster Group
   - From IBM Cloud Web Console
     > https://cloud.ibm.com/satellite/clusters -> Cluster groups -> Create cluster group
   - Add cluster to the group
     > https://cloud.ibm.com/satellite/groups -> select the cluster group -> Clusters -> Add cluster

4. View the cluster group from CLI
```
$ibmcloud sat group get --group test-group2               
OK
Cluster Group            
Name:                 test-group2   
ID:                   3a8ac73d-fee1-498a-88a1-d651db5152a5   
Created:              2021-02-09T12:36:18.971Z   
Cluster Count:        1   
Subscription Count:   0  

Subscriptions      
Name      ID                                     Version   

Clusters      
Name                          ID                                     Location   
satellite-ocs-template-test   b201d0ed-a4aa-414c-b0eb-0c4437797e95   c040tu4w0h6c6s5s9irg   
```

5. Create a storage configuration using the existing OCS template
   - Review the required parameters for the template
   - Create Storage Configuration (Sample provided below)


  We need to provide the disk-by-IDs of the disks we want to use to the `osd-device-path` and `mon-device-path` parameters as demonstrated in the example below
```
$ibmcloud sat storage config create --name ocs-config --template-name ocs --template-version 4.6_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part2" -p "mon-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part1" -p "num-of-osd=1" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90"
Creating Satellite storage configuration...
OK
Storage configuration 'ocs-config' was successfully created with ID 'b3982666-75a2-466d-9f0c-efc878dd5949'.
```

6. Deploy the configuration
   - Create assignment
 ```
$ ibmcloud sat storage assignment create --name ocs-sub --group test-group2 --config ocs-config
Creating assignment...
OK
Assignment ocs-sub was successfully created with ID 575cb060-b1ad-49fa-ab1a-7ce861fabc41.
```

#### Verifying your OCS storage configuration is assigned to your clusters

Note that the OCS installation takes about 15 minutes.

1. Get the status of your OCS storage cluster.

```
$ oc get storagecluster -n openshift-storage
NAME                 AGE   PHASE   EXTERNAL   CREATED AT             VERSION
ocs-storagecluster   72m   Ready              2021-02-10T06:00:20Z   4.6.0
```

2. List the OCS pods that are installed and verify that the status is `Running`.

```
$ oc get pods -n openshift-storage
NAME                                                              READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-9g2d5                                            3/3     Running     0          8m11s
csi-cephfsplugin-g42wv                                            3/3     Running     0          8m11s
csi-cephfsplugin-provisioner-7b89766c86-l68sr                     5/5     Running     0          8m10s
csi-cephfsplugin-provisioner-7b89766c86-nkmkf                     5/5     Running     0          8m10s
csi-cephfsplugin-rlhzv                                            3/3     Running     0          8m11s
csi-rbdplugin-8dmxc                                               3/3     Running     0          8m12s
csi-rbdplugin-f8c4c                                               3/3     Running     0          8m12s
csi-rbdplugin-nkzcd                                               3/3     Running     0          8m12s
csi-rbdplugin-provisioner-75596f49bd-7mk5g                        5/5     Running     0          8m12s
csi-rbdplugin-provisioner-75596f49bd-r2p6g                        5/5     Running     0          8m12s
noobaa-core-0                                                     1/1     Running     0          4m37s
noobaa-db-0                                                       1/1     Running     0          4m37s
noobaa-endpoint-7d959fd6fb-dr5x4                                  1/1     Running     0          2m27s
noobaa-operator-6cbf8c484c-fpwtt                                  1/1     Running     0          9m41s
ocs-operator-9d6457dff-c4xhh                                      1/1     Running     0          9m42s
rook-ceph-crashcollector-169.48.170.83-89f6d7dfb-gsglz            1/1     Running     0          5m38s
rook-ceph-crashcollector-169.48.170.88-6f58d6489-b9j49            1/1     Running     0          5m29s
rook-ceph-crashcollector-169.48.170.90-866b9d444d-zk6ft           1/1     Running     0          5m15s
rook-ceph-drain-canary-169.48.170.83-6b885b94db-wvptz             1/1     Running     0          4m41s
rook-ceph-drain-canary-169.48.170.88-769f8b6b7-mtm47              1/1     Running     0          4m39s
rook-ceph-drain-canary-169.48.170.90-84845c98d4-pxpqs             1/1     Running     0          4m40s
rook-ceph-mds-ocs-storagecluster-cephfilesystem-a-6dfbb4fcnqv9g   1/1     Running     0          4m16s
rook-ceph-mds-ocs-storagecluster-cephfilesystem-b-cbc56b8btjhrt   1/1     Running     0          4m15s
rook-ceph-mgr-a-55cc8d96cc-vm5dr                                  1/1     Running     0          4m55s
rook-ceph-mon-a-5dcc4d9446-4ff5x                                  1/1     Running     0          5m38s
rook-ceph-mon-b-64dc44f954-w24gs                                  1/1     Running     0          5m30s
rook-ceph-mon-c-86d4fb86-s8gdz                                    1/1     Running     0          5m15s
rook-ceph-operator-69c46db9d4-tqdpt                               1/1     Running     0          9m42s
rook-ceph-osd-0-6c6cc87d58-79m5z                                  1/1     Running     0          4m42s
rook-ceph-osd-1-f4cc9c864-fmwgd                                   1/1     Running     0          4m41s
rook-ceph-osd-2-dd4968b75-lzc6x                                   1/1     Running     0          4m40s
rook-ceph-osd-prepare-ocs-deviceset-0-data-0-29jgc-kzpgr          0/1     Completed   0          4m51s
rook-ceph-osd-prepare-ocs-deviceset-1-data-0-ckvv2-4jdx5          0/1     Completed   0          4m50s
rook-ceph-osd-prepare-ocs-deviceset-2-data-0-szmjd-49dd4          0/1     Completed   0          4m50s
rook-ceph-rgw-ocs-storagecluster-cephobjectstore-a-7f7f6df9rv6h   1/1     Running     0          3m44s
rook-ceph-rgw-ocs-storagecluster-cephobjectstore-b-554fd9dz6dm8   1/1     Running     0          3m41s
```

#### Scaling (Capacity expansion of OCS) :

**Important: Do not delete your storage configurations or assignments. Deleting configurations and assignments might result in data loss.**

##### Scaling by adding extra workers to the cluster :

To scale your OCS configuration, add worker nodes with local disks to your Satellite cluster.

To scale your OCS cluster, create a configuration with the same `ocs-cluster-name` and configuration details as the previous configuration, but increase the `num-of-osd` parameter value and specify the new worker node names with the `worker-nodes` parameter.

When you want to increase your storage capacity, you have to increase `num-of-osd` by the number of disks you add (taking into consideration the replication factor, which is `3` by default)

When you run the `sat storage config create` command, be sure to specify the following parameters to scale up your OCS cluster.

Enter the worker nodes where you want to install OCS and include the worker nodes that you added to your cluster.

Example :

```
$ibmcloud sat storage config create --name ocs-config2 --template-name ocs --template-version 4.6_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part2" -p "mon-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90,169.48.170.84,169.48.170.85,169.48.170.86"
```

After this, we need to create a new assignment for this configuration :

```
$ ibmcloud sat storage assignment create --name ocs-sub2 --group test-group2 --config ocs-config2
```

##### Scaling by either adding new disks to the existing workers or use exisitng disks already available on the worker nodes:

You scale your OCS configuration by adding disks to the worker nodes and providing the device paths of the new disks. Or, if your worker nodes already have extra local disks available, you can provide the device paths of those disks.

To retrieve the device details of your worker nodes, following the steps in the prerequisites.

To scale your OCS cluster, create a configuration with the same `ocs-cluster-name` and configuration details as the previous configuration, but increase the `num-of-osd` parameter value and add the new device paths.

In this example, the device path `/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part3` is the path of the disk that is added and the `num-of-osd` parameter is increased to 2.

```
$ibmcloud sat storage config create --name ocs-config2 --template-name ocs --template-version 4.6_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part2,/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part3" -p "mon-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90"
```

After this, we need to create a new assignment for this configuration :

```
$ ibmcloud sat storage assignment create --name ocs-sub2 --group test-group2 --config ocs-config2
```

#### How to update OCS version :

**Important: Do not delete your storage configurations or assignments. Deleting configurations and assignments might result in data loss.**

To update the OCS version of your configuration, get the details of your configuration and create a configuration with the same name and details, but set the `ocs-upgrade` parameter to `true`.

1. Get the details of your OCS configuration.
    ```
    ic sat storage config get <config-name>
    ```

2. Get the configuration details of your ocs-cluster custom resource.

```
kubectl get storagecluster <ocs-cluster-name>
```

3. Save the configuration details. When you upgrade your OCS version, you must enter the same configuration details and set the ocs-upgrade parameter to true.

Example :

```
$ibmcloud sat storage config create --name ocs-config3 --template-name ocs --template-version 4.7_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part2,/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part3" -p "mon-device-path=/dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ocs-upgrade=true"
```

4. Assign your configuration to your cluster groups.

```
$ ibmcloud sat storage assignment create --name ocs-sub3 --group test-group2 --config ocs-config3
```

This will upgrade OCS to the new version.

## Troubleshooting

You can check the CRD OcsCluster to see your parameters and storagcluster status

```
$ oc get ocscluster
apiVersion: ocs.ibm.io/v1
kind: OcsCluster
metadata:
  name: testocscluster
spec:
    billingType: hourly
    monDevicePaths:
    - /dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part1
    monSize: "1"
    monStorageClassName: localfile
    numOfOsd: 1
    ocsUpgrade: false
    osdDevicePaths:
    - /dev/scsi-3600605b00d87b43027b3bc310a64c6c9-part2
    osdSize: "1"
    osdStorageClassName: localblock
    workerNodes:
    - 169.48.170.83
    - 169.48.170.88
    - 169.48.170.90
status:
   storageClusterStatus: Ready
```

If the storageClusterStatus is stuck in `Progressing` or if it's `Error`, OCS installation has failed.

### If your OCS installation fails, complete the following the steps to troubleshoot your deployment.
- Check the describe of storagecluster and cephcluster in the openshift-storage namespace

```
$ oc describe storagecluster -n openshift-storage
$ oc describe cephcluster -n openshift-storage
```

- You can use the toolbox available in rook community to debug OCS deploy issues.
  Use this to install toolbox :
```
oc patch ocsinitialization ocsinit -n openshift-storage --type json --patch  '[{ "op": "replace", "path": "/spec/enableCephTools", "value": true }]'
```
- Use command `oc debug node/<Worker_Node>` and then run the command `lsblk` to check if your disks used in OCS are raw devices or raw partitions and they do not have any filesystems on them.
```
Sample output:
sh-4.2# lsblk -f
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda                                                      
|-sda1 xfs          9c0c6660-e1f4-45c8-a023-345da3d666c5 /boot
|-sda2 swap         6b5b21d1-e95e-45d9-b4fc-da805b997bec
`-sda3 xfs          f891eede-4ed8-42ec-8215-700d39afe119 /
sdb                                                      
`-sdb1 xfs          b4e4af05-56d7-4bc8-b235-32e540a51f1a /disk1
sdc                                                      
|-sdc1 ext4         cd7d288b-1ff3-4225-9465-ae6eef33deb5
|-sdc2 ext4         fd9367a3-cb73-4899-a202-0cd462c755a2
`-sdc3 ext4         d6874707-a2d4-489e-bfdb-521d6a8acbcb

```

## How to uninstall

1) Delete all the assignments and configurations created
2) Remove the finalizer on the OcsCluster resource and delete it
3) You'll have to first delete the `openshift-storage` namespace after removing finalizers on the resources under it
4) After that, you have to delete the `local-storage` namespace after removing finalizers on the resources under it
5) run this command for all the nodes to clean-up the files created by OCS.
```
oc debug node/<node name> -- chroot /host rm -rvf /var/lib/rook /mnt/local-storage
```
