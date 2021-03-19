# Red Hat OpenShift Container Storage - Local Storage

Red Hat OpenShift Container Storage is a software-defined storage that is optimised for container environments. It runs as an operator on OpenShift Container Platform to provide highly integrated and simplified persistent storage management for containers.

The user has to provide the input values to the custom resource OcsCluster while creating the satellite configuration to deploy OCS

## Prerequisites
In order to deploy OCS, the following prerequisites are required.
- [Create a Satellite location](cloud.ibm.com/docs/satellite?topic=satellite-locations).
- [Create a Satellite cluster](cloud.ibm.com/docs/satellite?topic=openshift-satellite-clusters).
- Your hosts must meet the [Satellite host requirements](https://cloud.ibm.com/docs/satellite?topic=satellite-host-reqs) in addition to having one of the following remote storage configurations. 
    * Two raw devices in block mode that have no partitions or formatted file systems. If your devices have no partitions, each node must have 2 free disks. One disk for the OSD and one disk for the MON.
    * Two raw partitions that have no formatted file system. If your raw devices are partitioned, they must have at least 2 partitions per disk, per worker node.
- Your cluster must have a minimum of 3 worker nodes with at least 16CPUs and 64GB RAM per worker node.
- Your cluster should be compatible with the OCS version that you're trying to install.
- [Add your Satellite to a cluster group](cloud.ibm.com/docs/satellite?topic=satellite-cluster-config#setup-clusters-satconfig-groups). 
- You must provision an instance of IBM Cloud Object Storage and provide your COS HMAC credentials, the regional public endpoint, and the IBM COS location when you create your storage configuration.

### Creating the IBM COS service instance

Run the following commands to create a COS instance and create a set of HMAC credentials.

1. Create an IBM COS Service Instance.
    ```
    ibmcloud resource service-instance-create noobaa-stor cloud-object-storage standard global
    ```

2. Create HMAC credentials. Make a note of your credentials.
    ```
    ibmcloud resource service-key-create cos-cred-rw Writer --instance-name noobaa-stor --parameters '{ "HMAC": true}'
    ```

3. Get the regional public endpoint and the location of your IBM COS instance. 
    You can get the endpoint details from the list of [IBM COS endpoints](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-endpoints) and the
    location from [IBM COS location constraint details](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-classes).


### Getting the device details for your OCS configuration.

When you create your OCS configuration, you must specify device paths for the object storage daemons (OSDs) and the MON. The OSD deploys a series of pods, in multiples of 3, that replicate your local storage across the worker nodes and disks that you configure for your {[ocs]} deployment. The device paths that you retrieve are specified as parameters when you create your OCS configuration.

1. Log in to your cluster and get a list of available worker nodes. Make a note of the worker nodes that you want to use in your OCS configuration.
    ```
    oc get nodes
    ```

2. Log in to each worker node that you want to use for your OCS configuration.
    ```
    oc debug node/<node-name>
    ```

3. When the debug pod is deployed on the worker node, run the following commands to list the available disks on the worker node.

    ```
    chroot /host && lsblk
    ```

4. Review the command output for available disks. Disks that can be used for your OCS configuration must be unmounted. In the following example output, the `sdc` disk has two available, unformatted partitions that we can use for the OSD and MON device paths for this worker node. As a best practice, and to maximize storage capacity on this disk, you can specify the smaller partition for the MON, and the larger partition for the OSD. Note that if you have raw disks with no partitions you need one disk for the OSD and one disk for the MON.  If your unmounted disks are partitioned, you can use the path of the partitions to find the disk-by-id.
    ```
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

5. Find the `by-id` for each disk that you want to use in your configuration. In this case, the `sdc1` and `sdc2` partitions are available. You can now find the disk `by-id`. The `by-id` for each disk is specified as a command parameter when you create your configuration.

    ```
    ls -l /dev/disk/by-id/
    ```

6. Review the command output and make a note of the `by-id` values that you want to use. In the following example output, you can see that the disk by ids for the `sdc1` and `sdc2` partitions are: `scsi-3600605b00d87b43027b3bc310a64c6c9-part1` and `scsi-3600605b00d87b43027b3bc310a64c6c9-part2`.
    ```
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

7. Repeat the previous steps for each worker node that you want to use for your OCS configuration.   

## Red hat Openshift Container Storage - Local Storage: Parameter reference

**Description of the template parameters :**

| Parameter | Required? | Description | Default value if not provided | Datatype |
| --- | --- | --- | --- | --- |
| `ocs-cluster-name` | Required | Enter the name of your OcsCluster custom resource. | N/A | string |
| `mon-device-path` | Required | Enter the `disk-by-id` paths to the devices that you want to use for the MON pods. Example: `/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part1`. | N/A | csv |
| `osd-device-path` | Required | Enter the `disk-by-id` paths to the devices that you want to use for the OSD pods. Example: `/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part2`. | N/A | csv |
| `num-of-osd` | Optional | Enter the number of OSDs. OCS will create 3x number of OSDs for the value specified. Initial storage capacity is the same as your disk size specified at `osd-device-path`. When you want to increase your storage capacity, you have to increase `num-of-osd` by the number of disks you add (taking into consideration the replication factor, which is `3` by default) | 1 | integer |
|`worker-nodes` | Optional | Enter the IP addresses of the worker nodes where you want to deploy OCS. If you do not specify the `worker-nodes`, OCS is installed on all of the worker nodes in your cluster. The minimum number of worker nodes that you must specify is 3. | N/A |csv |
| `billing-type` | Optional | Enter the billing option that you want to use. You can enter either `hourly` or `monthly`. | `hourly` | string |
| `ocs-upgrade` | Optional | Set to `true` if you want to upgrade the major version of OCS while creating a configuration of the newer version. | false | boolean |
| `ibm-cos-endpoint` | Required | Enter the IBM COS regional public endpoint. Example: `https://s3.us-east.cloud-object-storage.appdomain.cloud` | N/A | string |
| `ibm-cos-location` | Required | Enter the IBM COS regional location. Example: `us-east-standard` | N/A | string |
| `ibm-cos-access-key` | Required | Enter your IBM COS access key ID. | N/A | string |
| `ibm-cos-secret-key` | Required | Enter your IBM COS secret access key. | N/A | string |

## Default storage classes

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| sat-ocs-cephrbd-gold | ceph-rbd | ext4 | N/A | N/A | SSD | Delete |
| sat-ocs-cephfs-gold | cephfs |  N/A | N/A | N/A | SSD | Delete |
| sat-ocs-cephrgw-gold | ceph-rgw | N/A | N/A | N/A | SSD |Delete |
| sat-ocs-noobaa-gold | noobaa |  N/A | N/A | N/A | N/A | Delete |




## Creating the Red Hat Openshift Container Storage - Local storage configuration

1. Log in into your cluster using `oc` CLI or IBM Cloud CLI.
2. Verify that all the worker nodes have the `Ready` status.

    ```
    $oc get nodes
    NAME            STATUS   ROLES           AGE   VERSION
    169.48.170.83   Ready    master,worker   28h   v1.19.0+3b01205
    169.48.170.88   Ready    master,worker   28h   v1.19.0+3b01205
    169.48.170.90   Ready    master,worker   28h   v1.19.0+3b01205
    ```

3. Create a cluster group.
   - From the [IBM Cloud Web Console](https://cloud.ibm.com/satellite/clusters), click **Cluster groups** -> **Create cluster group**
   - Add your cluster to the group. From the [groups page](https://cloud.ibm.com/satellite/groups), select your cluster group and click **Clusters** -> **Add cluster.

4. Verify that your cluster group is created.
    ```
    ibmcloud sat group get --group test-group2  
    ```

    Example output:
    ```
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

5. Create a storage configuration using the existing OCS template. Enter the device details that your retrieved earlier. Be sure to provide the disk-by-IDs of the disks we want to use as the `osd-device-path` and `mon-device-path` [parameters]( https://github.com/IBM/ibm-satellite-storage/tree/master/config-templates/redhat/ocs-local/README.md##Prerequisites). Note that if your OCS configuration has 3 worker nodes, you must specify a total of 6 disks or partitions. 3 for the OSDs and 3 for the MONs.
    ```
    ibmcloud sat storage config create --name ocs-config --template-name ocs-local --template-version 4.6 -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part2,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part2,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part2" -p "mon-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part1,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part1,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part1" -p "num-of-osd=1" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
    ```

    Example output:
    ```
    Creating Satellite storage configuration...
    OK
    Storage configuration 'ocs-config' was successfully created with ID 'b3982666-75a2-466d-9f0c-efc878dd5949'.
    ```

## Creating the storage assignment

1. Run the following command to assign your OCS storage configuration to your cluster group. Note that the OCS installation takes about 15 minutes.
    ```
    ibmcloud sat storage assignment create --name ocs-sub --group test-group2 --config ocs-config
    ```
    Example output:
    ```
    Creating assignment...
    OK
    Assignment ocs-sub was successfully created with ID 575cb060-b1ad-49fa-ab1a-7ce861fabc41.
    ```

2. Verify that your OCS storage configuration is assigned to your clusters by getting the status of your OCS storage cluster.
    ```
    oc get storagecluster -n openshift-storage
    ```

    Example output:
    ```
    NAME                 AGE   PHASE   EXTERNAL   CREATED AT             VERSION
    ocs-storagecluster   72m   Ready              2021-02-10T06:00:20Z   4.6.0
    ```

3. List the OCS pods that are installed and verify that the status is `Running`.
    ```
    oc get pods -n openshift-storage
    ```
    Example output
    ```
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

## Scaling your OCS configuration:

**Important: Do not delete your storage configurations or assignments. Deleting configurations and assignments might result in data loss.**

You can scale your OCS configuration by addings worker nodes with unformatted disks to your clusters or by adding unformatted disks to your existing worker nodes.

**Example scaling by adding nodes**
Note that worker nodes must be added to your cluster in multiples of three. In this example the worker nodes that were created when the cluster was created each had 2 available unformatted disks. Worker nodes that are added to the cluster are added in multiples of 3 and each additional worker node that is added has at least 1 available unformatted disk or partition.

| Number of worker nodes | Total number of raw, unformatted disks or partitions | Total number of OSD devices | Total number of MON devices | `num-of-osd` |
| --- | --- | --- | --- | --- |
| 3 | 6 | 3 | 3 | 1 |
| 6 | 9 | 6 | 3 | 2 |
| 9 | 12 | 9 | 3| 3 |

**Example scaling by adding disks**
Note that disks must be added to your worker nodes in multiples of three. In this example, each original worker node has 2 available raw disks or partitions. 3 are used for the OSDs and 3 are used for the MON devices. When disks are added to the worker nodes, they must be added in multiples of 3. Since OCS requires only 3 MON devices, you do not need to add disks for the MON when you scale your OCS configuration. Any disks that are added are used as OSD devices.

| Number of worker nodes | Total number of raw, unformatted disks or partitions | Total number of OSD devices | Total number of MON devices | `num-of-osd` |
| --- | --- | --- | --- | --- |
| 3 | 6 | 3 | 3 | 1 |
| 3 | 9 | 6 | 3 | 2 |
| 3 | 12 | 9 | 3 | 3 |


### Scaling by adding worker nodes to your cluster

To scale your OCS configuration by adding worker nodes, create a storage configuration with the same `ocs-cluster-name` and configuration details as your existing configuration, but increase the `num-of-osd` parameter value and specify the new worker node names with the `worker-nodes` parameter.

In the following example, 3 worker nodes are added to the configuration that was created previously in the steps above. You can scale your configuration by adding updating the command parameters as follows:
    - `name` - Create a configuration with a new name.
    - `template-name` - This parameter remains the same as the existing configuration.
    - `template-version` - This parameter remains the same as the existing configuration.
    - `ocs-cluster-name` - This parameter remains the same as the existing configuration.
    - `osd-device-path` - Specify all previous `osd-device-path` values from your exisiting configuration as well as the device paths from the worker nodes that you have added to your cluster. You can retrieve the `by-id` values for your new worker nodes by reviewing the **Getting the details for your OCS configuration** on this page.
    - `mon-device-path` - Specify all previous `mon-device-path` values from your exisiting configuration. OCS requires 3 MON devices. You can retrieve the `by-id` values for your new worker nodes by reviewing the **Getting the details for your OCS configuration** on this page.
    - `num-of-osd` - Increase the OSD number by 1 for each set of 3 disks or partitions that you add to your configuration.
    - `worker-nodes` - Specify all of the worker nodes from your existing configuration plus any additonal worker nodes that you added to your cluster. Worker nodes must be added in multiples of 3.
    - `ibm-cos-endpoint` - This parameter remains the same as the existing configuration.
    - `ibm-cos-location` - This parameter remains the same as the existing configuration.
    - `ibm-cos-access-key` - This parameter remains the same as the existing configuration.
    - `ibm-cos-secret-key` - This parameter remains the same as the existing configuration.

1. Create the storage configuration and specify the updated values. In this example, the `osd-device-path` parameter is updated to include the device ids of the disks that you want to use. The worker node parameter is updated to include the worker nodes that are added to the cluster and the `num-of-osd` value is increased to 2.
    ```
    ibmcloud sat storage config create --name ocs-config2 --template-name ocs-local --template-version 4.6 -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part2,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part2,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part2" -p "mon-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part1,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part1,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90,169.48.170.84,169.48.170.85,169.48.170.86" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
    ```

2. Create a new assignment for this configuration.

    ```
    ibmcloud sat storage assignment create --name ocs-sub2 --group test-group2 --config ocs-config2
    ```


### Scaling by adding disks to the existing workers or use existing available disks on the worker nodes

To scale your OCS configuration by adding worker nodes, create a storage configuration with the same `ocs-cluster-name` and configuration details as your existing configuration, but increase the `num-of-osd` parameter value and specify the new worker node names with the `worker-nodes` parameter.

In the following example, 3 worker nodes are added to the configuration that was created previously in the steps above. You can scale your configuration by adding updating the command parameters as follows:
    - `name` - Create a configuration with a new name.
    - `template-name` - This parameter remains the same as the existing configuration.
    - `template-version` - This parameter remains the same as the existing configuration.
    - `ocs-cluster-name` - This parameter remains the same as the existing configuration.
    - `osd-device-path` - Specify all previous `osd-device-path` values from your exisiting configuration as well as the device paths from the worker nodes that you have added to your cluster. You can retrieve the `by-id` values for your new worker nodes by reviewing the **Getting the details for your OCS configuration** on this page.
    - `mon-device-path` - Specify all previous `mon-device-path` values from your exisiting configuration. OCS requires 3 MON devices. You can retrieve the `by-id` values for your new worker nodes by reviewing the **Getting the details for your OCS configuration** on this page.
    - `num-of-osd` - Increase the OSD number by 1 for each set of 3 disks or partitions that you add to your configuration.
    - `worker-nodes` - Specify the worker nodes from your existing configuration.
    - `ibm-cos-endpoint` - This parameter remains the same as the existing configuration.
    - `ibm-cos-location` - This parameter remains the same as the existing configuration.
    - `ibm-cos-access-key` - This parameter remains the same as the existing configuration.
    - `ibm-cos-secret-key` - This parameter remains the same as the existing configuration.

1. Create the storage configuration and specify the updated values. In this example, the `osd-device-path` parameter is updated to include the device ids of the disks that you want to use and the `num-of-osd` value is increased to 2.
    ```
    $ibmcloud sat storage config create --name ocs-config2 --template-name ocs-local --template-version 4.6 -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part2,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part2,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part2,/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part3,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part3,/dev/scsi-3600062b206ba6f00276eb58065b5da94-part3" -p "mon-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part1,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part1,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part1" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
    ```

2. Create a new assignment for this configuration :

    ```
    $ ibmcloud sat storage assignment create --name ocs-sub2 --group test-group2 --config ocs-config2
    ```

## Upgrading your OCS version :

**Important: Do not delete your storage configurations or assignments. Deleting configurations and assignments might result in data loss.**

To upgrade the OCS version of your configuration, get the details of your configuration and create a new configuration with the same `ocs-cluster-name` and details, but with the `template-version` set to the version you want to upgrade to and set the `ocs-upgrade` parameter to `true`.

In the following example, the OCS configuration is updated to use template version 4.7:
    - `name` - Enter a name for your new configuration.
    - `template-name` - This parameter remains the same as the existing configuration.
    - `template-version` - Enter the template version that you want to use to upgrade your configuration.
    - `ocs-cluster-name` - This parameter remains the same as the existing configuration.
    - `osd-device-path` - This parameter remains the same as the existing configuration.
    - `mon-device-path` - This parameter remains the same as the existing configuration.
    - `num-of-osd` - This parameter remains the same as the existing configuration.
    - `worker-nodes` - This parameter remains the same as the existing configuration.
    - `ibm-cos-endpoint` - This parameter remains the same as the existing configuration.
    - `ibm-cos-location` - This parameter remains the same as the existing configuration.
    - `ibm-cos-access-key` - This parameter remains the same as the existing configuration.
    - `ibm-cos-secret-key` - This parameter remains the same as the existing configuration.
    - `ocs-upgrade` - Enter `true` to upgrade your `ocs-cluster` to the template version that you specified.

1. Get the details of your OCS configuration.
    ```
    ic sat storage config get <config-name>
    ```

2. Get the configuration details of your ocs-cluster custom resource.

    ```
    kubectl get ocscluster <ocs-cluster-name>
    ```

3. Save the configuration details. When you upgrade your OCS version, you must enter the same configuration details and set the `template-version` to the version you want to upgrade to and set the `ocs-upgrade` parameter to `true`.
    ```
    $ibmcloud sat storage config create --name ocs-config3 --template-name ocs-local --template-version 4.7 -p "ocs-cluster-name=testocscluster" -p "osd-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part2,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part2,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part2" -p "mon-device-path=/dev/disk/by-id/scsi-3600605b00d87b43027b3bc310a64c6c9-part1,/dev/disk/by-id/scsi-3600605b00d87b43027b3bbf306bc28a7-part1,/dev/disk/by-id/scsi-3600062b206ba6f00276eb58065b5da94-part1" -p "num-of-osd=1" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy" -p "ocs-upgrade=true"
    ```

4. Assign your configuration to your cluster groups.

    ```
    $ ibmcloud sat storage assignment create --name ocs-sub3 --group test-group2 --config ocs-config3
    ```

5. Verify that your configuration is updated.
    ```
    ic sat storage config get <config-name>
    ```


## Troubleshooting

Check the status of your storage cluster.
    ```
    oc get ocscluster
    ```

   Example output

    ```
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

If the `storageClusterStatus` is `Progressing` or `Error`, the OCS installation has failed.

### If your OCS installation fails, complete the following the steps to troubleshoot your deployment.
1. Check the describe of storagecluster and cephcluster in the openshift-storage namespace and look at the `Events` and the `Status` sections

    ```
    oc describe storagecluster -n openshift-storage
    oc describe cephcluster -n openshift-storage
    ```

2. You can use the toolbox available in rook community to debug OCS deploy issues. Run the following command to install the toolbox.
    ```
    oc patch ocsinitialization ocsinit -n openshift-storage --type json --patch  '[{ "op": "replace", "path": "/spec/enableCephTools", "value": true }]'
    ```

3. Verify that the disks or partitions that you specified in your OCS configuration are unmounted and unformatted. Review the steps in the **Prerequisites** section to check your device details.

## Removing your OCS configuration

1. List your storage assignments and find the one that you used for your cluster. 
    ```
    ic sat storage assignment ls
    ```
2. Remove the assignment. After the assignment is removed, the driver pods and storage classes are removed from all clusters that were part of the storage assignment.
    ```
    ic sat storage assignment rm --assignment <assignment_name>
    ```

3. Remove your storage configuration.
    ```
    ic sat storage config rm --config <config_name>
    ```
4. Clean up the remaining Kubernetes resources from your cluster. Save the following script in a file called `cleanup.sh` to your local machine.
    ```
    #!/bin/bash
    oc delete ns openshift-storage --wait=false
    sleep 20
    kubectl -n openshift-storage patch persistentvolumeclaim/db-noobaa-db-0 -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch cephblockpool.ceph.rook.io/ocs-storagecluster-cephblockpool -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch cephcluster.ceph.rook.io/ocs-storagecluster-cephcluster -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch cephfilesystem.ceph.rook.io/ocs-storagecluster-cephfilesystem -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch cephobjectstore.ceph.rook.io/ocs-storagecluster-cephobjectstore -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch cephobjectstoreuser.ceph.rook.io/noobaa-ceph-objectstore-user -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch cephobjectstoreuser.ceph.rook.io/ocs-storagecluster-cephobjectstoreuser -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch NooBaa/noobaa -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch backingstores.noobaa.io/noobaa-default-backing-store -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch bucketclasses.noobaa.io/noobaa-default-bucket-class -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n openshift-storage patch storagecluster.ocs.openshift.io/ocs-storagecluster -p '{"metadata":{"finalizers":[]}}' --type=merge
    sleep 20
    oc delete pods -n openshift-storage --all --force --grace-period=0
    oc delete ns local-storage --wait=false
    sleep 20
    kubectl -n local-storage patch localvolume.local.storage.openshift.io/local-block -p '{"metadata":{"finalizers":[]}}' --type=merge
    kubectl -n local-storage patch localvolume.local.storage.openshift.io/local-file -p '{"metadata":{"finalizers":[]}}' --type=merge
    sleep 20
    oc delete pods -n local-storage --all --force --grace-period=0
    ```
5. Run the cleanup.sh script.
    ```
    sh ./cleanup.sh
    ```
6. After you run the cleanup script, log in to each worker node and run the following commands.
- Deploy a debug pod and run chroot /host.
    `oc debug node/<node name> -- chroot /host`

- Run the following command to remove any files or directories on the specified paths.
    `rm -rvf /var/lib/rook /mnt/local-storage`
