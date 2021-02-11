# OCS

Red Hat OpenShift Container Storage is software-defined storage that is optimised for container environments. It runs as an operator on OpenShift Container Platform to provide highly integrated and simplified persistent storage management for containers.

The user has to provide the input values to the custom resource OcsCluster while creating the satellite configuration to deploy OCS

##How to use the template to deploy OCS (local) on a Satellite Cluster?

### Prerequisites

1) In order to deploy OCS, at least one of these local storage options are required:
    - Raw devices (no partitions or formatted filesystems)
    - Raw partitions (no formatted filesystem)
    - PVs available from a storage class in block mode
2) The cluster needs to have a minimum of 3 nodes
3) The OCP version should be compatible with the OCS version you're trying to install  

### OCS local: Parameters

**Description of the template parameters :**

| **Parameter Name** |**Description** |**Datatype** |**Mandatory** |**Default Value** |
|--------------------|----------------|-------------|--------------|------------------|
|ocs-cluster-name | Name of the CRD (OcsCluster)   |string |Yes   |                  |
|mon-device-path      |Devices present on worker to be used for mon pods|csv |Yes |                  |
|osd-device-path      |Devices present on worker to be used for OSD pods  | csv|Yes |                  |
|num-of-osd        |Number of OSDs required. OCS will create 3x number of OSDs for the value specified here |integer |No |1 |
|worker-nodes         |Workers which need to be a part of OCS (Minimum 3). If not specified, all workers will be considered for OCS |csv |No |        |
|billing-type         |Billing option |String |No    |hourly            |
|ocs-upgrade          |Upgrade the OCS major version |boolean |No   |false            |
|ibm-cos-access-key         | IBM COS access key for using COS as default backend storage|string |No   |           |
|ibm-cos-secret-key          | IBM COS secret key for using COS as default backend storage|string |No   |            |

**Default storage classes**

| **Storage class name** | **Type** | **Provisioner** | **Reclaim policy**|
|------------------------|-----------------|--------|----------|
| sat-ocs-cephrbd-gold | Block | openshift-storage.rbd.csi.ceph.com | Delete |
| sat-ocs-cephfs-gold | File | openshift-storage.cephfs.csi.ceph.com | Delete |
| sat-ocs-cephrgw-gold | Object | openshift-storage.ceph.rook.io/bucket |Delete |
| sat-ocs-noobaa-gold | Object bucket | openshift-storage.noobaa.io/obc | Delete |

### Creating the OCS storage configuration

#### Detailed steps

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

5. Create storage configuration using existing template
   - Review the required parameters for the template
   - Create Storage Configuration (Sample provided below)
```
$ibmcloud sat storage config create --name ocs-config --template-name ocs --template-version 4.6_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/sdc2" -p "mon-device-path=/dev/sdc1" -p "num-of-osd=1" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
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

### Verifying your OCS storage configuration is assigned to your clusters

It takes about 15 minutes for OCS to get installed.
You can check the CRD OcsCluster to see your parameters and storagcluster status

```
$ oc get ocscluster
apiVersion: ocs.ibm.io/v1
kind: OcsCluster
metadata:
  name: testocscluster
spec:
    billingType: hourly
    ibmCosAccessKey: ""
    ibmCosSecretKey: ""
    monDevicePaths:
    - /dev/sdc1
    monSize: "1"
    monStorageClassName: localfile
    numOfOsd: 1
    ocsUpgrade: false
    osdDevicePaths:
    - /dev/sdc2
    osdSize: "1"
    osdStorageClassName: localblock
    workerNodes:
    - 169.48.170.83
    - 169.48.170.88
    - 169.48.170.90
status:
   storageClusterStatus: Ready
```

You can check the status of the storagecluster

```
$ oc get storagecluster -n openshift-storage
NAME                 AGE   PHASE   EXTERNAL   CREATED AT             VERSION
ocs-storagecluster   72m   Ready              2021-02-10T06:00:20Z   4.6.0
```

The deployment will install the following operators and pods inside the cluster

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

###Scaling (Capacity expansion of OCS) :

To scale the OCS cluster, we have to create a new configuration with the same name for the CRD (the parameter ocs-cluster-name) and the rest of the parameters should remain the same as the previous configuration, but, we should change the value of `num-of-osd` with the scaling factor required.
If no extra devices are present, we need to add devices by updating the parameter `osd-device-path`

Example :

```
$ibmcloud sat storage config create --name ocs-config2 --template-name ocs --template-version 4.6_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/sdc2,/dev/sdc3" -p "mon-device-path=/dev/sdc1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
```

After this, we need to create a new assignment for this configuration :

```
$ ibmcloud sat storage assignment create --name ocs-sub2 --group test-group2 --config ocs-config2
```

This will update the OcsCluster resource.

###Migration (Updating OCS version) :

To update the version of OCS, we have to create a new configuration with the template version set to the version you want to upgrade to. We need to have the same name for the CRD (the parameter ocs-cluster-name) and the rest of the parameters should remain the same as the previous configuration, but, we should change the value of `ocs-upgrade` to true.

Example :

```
$ibmcloud sat storage config create --name ocs-config3 --template-name ocs --template-version 4.7_local -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/sdc2,/dev/sdc3" -p "mon-device-path=/dev/sdc1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy" -p "ocs-upgrade=true"
```

After this, we need to create a new assignment for this configuration :

```
$ ibmcloud sat storage assignment create --name ocs-sub3 --group test-group2 --config ocs-config3
```

This will upgrade OCS to the new version.

## Troubleshooting
### What to do when OCS install fails
- Check the describe of storagecluster and cephcluster in openshift-storage namespace

```
$ oc describe storagecluster -n openshift-storage
$ oc describe cephcluster -n openshift-storage
```

- You can use the toolbox available in rook community to debug OCS deploy issues. Refer https://rook.io/docs/rook/v1.3/ceph-toolbox.html to use toolbox.
- Use command `oc debug node/<Worker_Node>` and then run the command `lsblk` to check if your disks used in OCS are raw devices or raw partitions and they do not have any filesystems on them.
```
Sample output:
lsblk -f
NAME                  FSTYPE      LABEL UUID                                   MOUNTPOINT
vda
└─vda1                LVM2_member       eSO50t-GkUV-YKTH-WsGq-hNJY-eKNf-3i07IB
  ├─ubuntu--vg-root   ext4              c2366f76-6e21-4f10-a8f3-6776212e2fe4   /
  └─ubuntu--vg-swap_1 swap              9492a3dc-ad75-47cd-9596-678e8cf17ff9   [SWAP]
vdb
```

## How to uninstall

- Delete the assignment and the configuration
- run this command for all the nodes
```
oc debug node/<node name> -- chroot /host rm -rvf /var/lib/rook /mnt/local-storage
```
