# Red Hat OpenShift Container Storage - Remote Storage

Red Hat OpenShift Container Storage is a software-defined storage that is optimised for container environments. It runs as an operator on OpenShift Container Platform to provide highly integrated and simplified persistent storage management for containers.

The user has to provide the input values to the custom resource OcsCluster while creating the satellite configuration to deploy OCS

## Prerequisites
In order to deploy OCS, the following prerequisites are required.
- [Create a Satellite location](cloud.ibm.com/docs/satellite?topic=satellite-locations).
- [Create a Satellite cluster](cloud.ibm.com/docs/satellite?topic=openshift-satellite-clusters).
- Your cluster must have a minimum of 3 worker nodes with at least 16CPUs and 64GB RAM per worker node.
- Your hosts must meet the [Satellite host requirements](https://cloud.ibm.com/docs/satellite?topic=satellite-host-reqs) in addition to having one of the following remote storage configurations.
  * Remote storage available in block mode
- [Add your Satellite to a cluster group](cloud.ibm.com/docs/satellite?topic=satellite-cluster-config#setup-clusters-satconfig-groups).
- Your cluster should be compatible with the OCS version that you're trying to install.
- The storage class you use for the `mon-storage-class` and `osd-storage-class` parameters should have `VolumeBindingMode` set to `WaitForFirstConsumer` if you're using a multizone cluster
- [Optional] You must provision an instance of IBM Cloud Object Storage and provide your COS HMAC credentials, the regional public endpoint, and the IBM COS location when you create your storage configuration if you want to use ibm cos as the default backingstore. If not provided, pv-pool will be used.

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
    You can get the endpoint details from the list of [IBM COS endpoints](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-endpoints) and the location from [IBM COS location constraint details](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-classes).

## Red hat Openshift Container Storage - Remote Storage: Parameter reference

**Description of the template parameters :**

| Parameter | Required? | Description | Default value if not provided | Datatype |
| --- | --- | --- | --- | --- |
| `ocs-cluster-name` | Required | Enter the name of your OcsCluster custom resource. | N/A | string |
| `mon-storage-class` | Required | Enter the storage class name that you want to use for the mon pods. The storage class must have the `waitForFirstConsumer` volume binding mode. | N/A | string |
| `mon-size` | Required | Enter the size of the mon pods | 20Gi | string |
| `osd-storage-class` | Required | Enter the storage class name that you want to use for the OSD pods. The storage class must have the `waitForFirstConsumer` volume binding mode.  | N/A | string |
| `osd-size` | Required | Enter the size of the osd pods | 100Gi | string |
| `num-of-osd` | Optional | Enter the number of OSDs. OCS will create 3x number of OSDs for the value specified. Initial storage capacity is the same as your osd size specified at `osd-size`. When you want to increase your storage capacity, you have to increase `num-of-osd` by the multiples of `osd-size` | 1 | integer |
|`worker-nodes` | Optional | Enter the IP addresses of the worker nodes where you want to deploy OCS. If you do not specify the `worker-nodes`, OCS is installed on all of the worker nodes in your cluster. The minimum number of worker nodes that you must specify is 3. | N/A |csv |
| `billing-type` | Optional | Enter the billing option that you want to use. You can enter either `hourly` or `monthly`. | `hourly` | string |
| `ocs-upgrade` | Optional | Set to `true` if you want to upgrade the major version of OCS while creating a configuration of the newer version. | false | boolean |
| `ibm-cos-endpoint` | Optional | Enter the IBM COS regional public endpoint. Example: `https://s3.us-east.cloud-object-storage.appdomain.cloud` | N/A | string |
| `ibm-cos-location` | Optional | Enter the IBM COS regional location. Example: `us-east-standard` | N/A | string |
| `ibm-cos-access-key` | Optional | Enter your IBM COS access key ID. | N/A | string |
| `ibm-cos-secret-key` | Optional | Enter your IBM COS secret access key. | N/A | string |

## Default storage classes

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| sat-ocs-cephrbd-gold | ceph-rbd | ext4 | N/A | N/A | SSD | Delete |
| sat-ocs-cephfs-gold | cephfs |  N/A | N/A | N/A | SSD | Delete |
| sat-ocs-cephrgw-gold | ceph-rgw | N/A | N/A | N/A | SSD |Delete |
| sat-ocs-noobaa-gold | noobaa |  N/A | N/A | N/A | N/A | Delete |




## Creating the Red Hat Openshift Container Storage - Remote storage configuration

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

5. Create a storage configuration using the existing OCS template.
    ```
    ibmcloud sat storage config create --name ocs-config --template-name ocs-remote --template-version 4.6 -p "ocs-cluster-name=testocscluster" -p "mon-storage-class=vpc-custom-10iops-tier" -p "mon-size=50Gi" -p "osd-storage-class=vpc-custom-10iops-tier" -p "osd-size=150Gi" -p "num-of-osd=1" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
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
    NAME                 AGE     PHASE   EXTERNAL   CREATED AT             VERSION
    ocs-storagecluster   5m14s   Ready              2021-03-02T11:46:41Z   4.6.0
    ```

3. List the OCS pods that are installed and verify that the status is `Running`.
    ```
    oc get pods -n openshift-storage
    ```
    Example output
    ```       
    NAME                                                              READY   STATUS      RESTARTS   AGE
    csi-cephfsplugin-9lp57                                            3/3     Running     0          5m34s
    csi-cephfsplugin-k684x                                            3/3     Running     0          5m34s
    csi-cephfsplugin-l75qk                                            3/3     Running     0          5m34s
    csi-cephfsplugin-provisioner-5cbd6fbc4f-c6cs2                     6/6     Running     0          5m34s
    csi-cephfsplugin-provisioner-5cbd6fbc4f-txswk                     6/6     Running     0          5m34s
    csi-rbdplugin-provisioner-97bdb4f66-6x9gp                         6/6     Running     0          5m34s
    csi-rbdplugin-provisioner-97bdb4f66-xqbs4                         6/6     Running     0          5m35s
    csi-rbdplugin-stx76                                               3/3     Running     0          5m35s
    csi-rbdplugin-wts6z                                               3/3     Running     0          5m35s
    csi-rbdplugin-zjfdd                                               3/3     Running     0          5m35s
    noobaa-core-0                                                     1/1     Running     0          3m2s
    noobaa-db-0                                                       1/1     Running     0          3m2s
    noobaa-endpoint-787c7c49f7-nwt44                                  1/1     Running     0          87s
    noobaa-operator-6d6b46745d-tdhsz                                  1/1     Running     0          6m3s
    ocs-metrics-exporter-5557759dd8-nnq56                             1/1     Running     0          6m2s
    ocs-operator-d67758886-xvgwv                                      1/1     Running     0          6m4s
    rook-ceph-crashcollector-172.33.27.124-5b99b6965d-ft6h7           1/1     Running     0          3m52s
    rook-ceph-crashcollector-172.33.41.67-7c9c4b9948-2wjg6            1/1     Running     0          4m13s
    rook-ceph-crashcollector-172.33.9.87-748568c884-ps68w             1/1     Running     0          4m29s
    rook-ceph-mds-ocs-storagecluster-cephfilesystem-a-5564549b72vdh   1/1     Running     0          2m48s
    rook-ceph-mds-ocs-storagecluster-cephfilesystem-b-7d4b4bd5p88gb   1/1     Running     0          2m47s
    rook-ceph-mgr-a-5645b47856-j6d4b                                  1/1     Running     0          3m33s
    rook-ceph-mon-a-6554b79495-ghpmk                                  1/1     Running     0          4m29s
    rook-ceph-mon-b-5bb99f8856-b8p4f                                  1/1     Running     0          4m13s
    rook-ceph-mon-c-779b7b5cdf-t6vkd                                  1/1     Running     0          3m52s
    rook-ceph-operator-7b8d579586-r8x27                               1/1     Running     0          6m3s
    rook-ceph-osd-0-7d44c55748-zddkn                                  1/1     Running     0          3m16s
    rook-ceph-osd-1-75d95b56dc-sjm26                                  1/1     Running     0          3m4s
    rook-ceph-osd-2-685686fdc8-vlb6c                                  1/1     Running     0          3m3s
    rook-ceph-osd-prepare-ocs-deviceset-0-data-0-mjf4n-nj6tk          0/1     Completed   0          3m31s
    rook-ceph-osd-prepare-ocs-deviceset-1-data-0-wpqnq-zdp6n          0/1     Completed   0          3m31s
    rook-ceph-osd-prepare-ocs-deviceset-2-data-0-bmrdz-zqngk          0/1     Completed   0          3m31s
    rook-ceph-rgw-ocs-storagecluster-cephobjectstore-a-54c8cdc2rxsd   1/1     Running     0          2m19s
    rook-ceph-rgw-ocs-storagecluster-cephobjectstore-b-5ddd9d546mcc   1/1     Running     0          2m18s
    ```

## Scaling your OCS configuration:

**Important: Do not delete your storage configurations or assignments. Deleting configurations and assignments might result in data loss.**

You can scale your OCS configuration by increasing the `num-of-osd` parameter.

**Note :** You can only scale in increments of `osd-size` provided by you in the initial storage configuration.

| Initial OSD size | `num-of-osd` | Storage capacity |
| --- | --- | --- |
| 150Gi | 1 | 150Gi |
| 150Gi | 2 | 300Gi |
| 150Gi | 3 | 450Gi |
| 200Gi | 1 | 200Gi |
| 200Gi | 2 | 400Gi |
| 200Gi | 3 | 600Gi |

1. Create the storage configuration and specify the updated values. In this example, the `num-of-osd` parameter is updated to 2, to double the storage capacity.
  ```
  ibmcloud sat storage config create --name ocs-config --template-name ocs-remote --template-version 4.6 -p "ocs-cluster-name=testocscluster" -p "mon-storage-class=vpc-custom-10iops-tier" -p "mon-size=50Gi" -p "osd-storage-class=vpc-custom-10iops-tier" -p "osd-size=150Gi" -p "num-of-osd=2" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
  ```

2. Create a new assignment for this configuration.

    ```
    ibmcloud sat storage assignment create --name ocs-sub2 --group test-group2 --config ocs-config2
    ```

## Upgrading your OCS version :

**Important: Do not delete your storage configurations or assignments. Deleting configurations and assignments might result in data loss.**

To upgrade the OCS version of your configuration, get the details of your configuration and create a new configuration with the same `ocs-cluster-name` and details, but with the `template-version` set to the version you want to upgrade to and set the `ocs-upgrade` parameter to `true`.

In the following example, the OCS configuration is updated to use template version 4.7:
    - `name` - Enter a name for your new configuration.
    - `template-name` - This parameter remains the same as the existing configuration.
    - `template-version` - Enter the template version that you want to use to upgrade your configuration.
    - `ocs-cluster-name` - This parameter remains the same as the existing configuration.
    - `mon-storage-class` - This parameter remains the same as the existing configuration.
    - `mon-size` - This parameter remains the same as the existing configuration.
    - `osd-storage-class` - This parameter remains the same as the existing configuration.
    - `osd-size` - This parameter remains the same as the existing configuration.
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
  ibmcloud sat storage config create --name ocs-config --template-name ocs-remote --template-version 4.7 -p "ocs-cluster-name=testocscluster" -p "mon-storage-class=vpc-custom-10iops-tier" -p "mon-size=50Gi" -p "osd-storage-class=vpc-custom-10iops-tier" -p "osd-size=150Gi" -p "num-of-osd=1" -p "worker-nodes=169.48.170.83,169.48.170.88,169.48.170.90" -p "ocs-upgrade=true" -p "ibm-cos-endpoint=https://s3.us-east.cloud-object-storage.appdomain.cloud" -p "ibm-cos-location=us-east-standard" -p "ibm-cos-access-key=xxx" -p "ibm-cos-secret-key=yyy"
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
    oc describe ocscluster
    ```

   Example output

    ```
    apiVersion: ocs.ibm.io/v1
    kind: OcsCluster
    metadata:
      name: testocscluster
    Spec:
      Billing Type: hourly
      Mon Size: 50Gi
      Mon Storage Class Name: vpc-custom-10iops-tier
      Num Of Osd: 2
      Ocs Upgrade: false
      Osd Size: 150Gi
      Osd Storage Class Name: vpc-custom-10iops-tier
      Status:
      Storage Cluster Status:  Ready
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
    ocscluster_name=`oc get ocscluster | awk 'NR==2 {print $1}'`
    oc delete ocscluster --all --wait=false
    kubectl patch ocscluster/$ocscluster_name -p '{"metadata":{"finalizers":[]}}' --type=merge
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
    ```
5. Run the cleanup.sh script.
    ```
    sh ./cleanup.sh
    ```
6. After you run the cleanup script, log in to each worker node and run the following commands.
- Deploy a debug pod and run chroot /host.
    `oc debug node/<node name> -- chroot /host`

- Run the following command to remove any files or directories on the specified paths.
    `rm -rvf /var/lib/rook`
