# Deploy local volume file storage on a Satellite Cluster
Local persistent volumes allow you to access local storage devices, such as a disk or partition, by using the standard persistent volume claim interface.

## Prerequisites
1. Attach additional free disk to the cluster worker nodes(hosts)

2. Login into the Cluster using `OC CLI`
   - Login to [IBM Cloud Web Console](https://cloud.ibm.com/)
	 - Go to [Clusters page](https://cloud.ibm.com/satellite/clusters)
   - Click on the row with the cluster details, You will get your cluster page
   - Click on `Manage Cluster` and in next page click on `OpenShift web console`
   - In OpenShift console click on your user ID at top right corner and then click on `Copy Login Command`
   - In next page click on `Display Token`
   - Under `Log in with this token` the `oc login --token=XXXX ...` will be displayed, copy the command and execute on your local system

    **Note** The target cluster version should be 4.12.X to use the local-volume-block version 4.10 template.

3. Add label to the worker nodes, one with additional disk
   - Get the nodes
     ```
     $ oc get nodes
     NAME           STATUS   ROLES           AGE   VERSION
     52.117.85.12   Ready    master,worker   20d   v1.18.3+fa69cae
     52.117.85.3    Ready    master,worker   20d   v1.18.3+fa69cae
     52.117.85.9    Ready    master,worker   20d   v1.18.3+fa69cae
     ```
   - Add label to nodes
     ```
     $ oc label nodes 52.117.85.9 52.117.85.12 "storage=localfile"
     node/52.117.85.9 labeled
     node/52.117.85.12 labeled
     ```

4. Login using IBM Cloud CLI
   - Login
     ```
     $ ibmcloud login --sso -a https://cloud.ibm.com -r us-east -g default -u <IBMid>
     ```
   - Target to regional API
     ```
     $ ibmcloud cs init --host https://us-east.containers.cloud.ibm.com
     ```
     - US-East: https://us-east.containers.cloud.ibm.com
     - London:  https://eu-gb.containers.cloud.ibm.com
     - US-South: https://us-south.containers.cloud.ibm.com

5. Create Cluster Group (optional)
   - From IBM Cloud Web Console
     > https://cloud.ibm.com/satellite/clusters -> Cluster groups -> Create cluster group
   - Add cluster to the group
     > https://cloud.ibm.com/satellite/groups -> select the cluster group -> Clusters -> Add clusters


## Local Volume parameters & how to retrieve them

Parameter | Required? | Description | Default value if not provided | 
----------| ----------| ------------| ----------------------------- |
`label-key` | Required  | You can retrieve this parameter by using ```$ oc get nodes --show-labels``` command | N/A | 
`label-value`| Required | You can retrieve this parameter by using ```$ oc get nodes --show-labels``` command | N/A | 
`devicepath` | Required | You can retrieve this parameter by using following commands ```$ oc get nodes``` ```$ oc debug node/<node name> ``` ```# chroot /host``` ```# lsblk``` | N/A | 
`fstype`     | Required | File System type of your choice     |          ext4|


## Default storage classes

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| ------------------ | ---- | ----------- | ---- | ---------- | ----------| -------------- |
| sat-local-file-gold| File | ext4/xfs    | N/A  | N/A        | N/A       | Retain         | 


## Creating the Local Volume File storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name localvol-file-config --template-name local-volume-file --template-version 4.12 -p "label-key=storage" -p "label-value=localfile" -p "devicepath=/dev/xvde"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name localvol-file-assign --cluster <clusterID> --config localvol-file-config
```

```sh
ibmcloud sat storage assignment create --name localvol-file-assign --group satClusterGrp --config localvol-file-config
```

## Verifying your Local Volume File storage configuration is assigned to your clusters

```
$ oc get all -n local-storage
NAME                                         READY   STATUS    RESTARTS   AGE
pod/diskmaker-discovery-lh5vt                2/2     Running   0          86s
pod/diskmaker-manager-c8gh4                  2/2     Running   0          86s
pod/local-storage-operator-f4967bf5d-gbg8q   1/1     Running   0          4m11s

NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/local-storage-discovery-metrics   ClusterIP   172.21.85.51    <none>        8383/TCP   87s
service/local-storage-diskmaker-metrics   ClusterIP   172.21.75.241   <none>        8383/TCP   87s

NAME                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/diskmaker-discovery   1         1         1       1            1           <none>          87s
daemonset.apps/diskmaker-manager     1         1         1       1            1           <none>          87s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/local-storage-operator   1/1     1            1           4m12s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/local-storage-operator-f4967bf5d   1         1         1       4m13s

```

```
$ oc get sc -n local-storage | grep local
sat-local-file-gold       kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  21m
```

```
$ oc get pv
NAME               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS          REASON   AGE
local-pv-1d14680   50Gi       RWO            Delete           Available           sat-local-file-gold            50s
```

**Follow** the [link](https://docs.openshift.com/container-platform/4.12/storage/persistent_storage/persistent-storage-local.html) to create the persistent volume claim and attach the claim to a pod.


## Troubleshooting

**Local PV creation fails**
   - Check for razeedeploy
     ```
     oc get ns
     ```
     ```
     oc get rr -n razeedeploy
     ```
   - Go to the IBM cloud web console.
1. Navigate to your cluster in the [IBM Cloud console](https://cloud.ibm.com/satellite/clusters)
2. Select your cluster and click on the overflow menu.
3. Click **Re-attach** cluster.
4. Copy the reattach command and run it in your command line.
5. Get the namespace where you deployed the local storage template and verify that the status is `Active`.
 ```sh
 oc get ns | grep local
 ```
 **Example output**
 ```
 local-storage                                      Active   101m
 ```

6. Get the logs for the `diskmaker-manager` pod to verify any mount issues or verify if PV creation is successful.
 ```sh
 kubectl logs -f pod/diskmaker-manager-nxpfr -c diskmaker-manager -n local-storage
 ```

 **Example output:**

```
kubectl logs -f pod/diskmaker-manager-nxpfr -c diskmaker-manager -n local-storage
I1028 07:43:47.868023   91852 diskmaker.go:21] Go Version: go1.18.4
I1028 07:43:47.868137   91852 diskmaker.go:22] Go OS/Arch: linux/amd64
I1028 07:43:47.868142   91852 diskmaker.go:23] local-storage-diskmaker Version: v4.12.0-202210061001.p0.g7670056.assembly.stream-0-g1a3cbf8-dirty
I1028 07:43:48.918719   91852 request.go:601] Waited for 1.033213982s due to client-side throttling, not priority and fairness, request: GET:https://172.21.0.1:443/apis/local.storage.openshift.io/v1?timeout=32s
I1028 07:43:50.473849   91852 common.go:408] Creating client using in-cluster config
E0213 06:19:40.697628       1 diskmaker.go:203] failed to acquire lock on device /dev/xvde
E0213 06:19:40.697657       1 diskmaker.go:180] error symlinking /dev/xvde to /mnt/local-storage/sat-local-file-gold/xvde: error acquiring exclusive lock on /dev/xvde
```

7. Log-in to the node where the diskmaker pod is deployed.
 ```sh
 oc debug node/<node-name>
 ```

8. Run the following command to use host binaries.
  ```sh
  chroot /host
  ```
9. Delete the symlink.
  ```sh
  rm -rf </path/to/symlink/as/shown/in/logs>
  ```

 **Example command:**
 ```sh
  rm -rf  /mnt/local-storage/sat-local-file-gold/xvde
  ```

10. Get the `diskmaker-discovery` pod logs to verify auto-discovery of devices if enabled.
    ```
     kubectl logs -f diskmaker-discovery-lh5vt -c diskmaker-discovery -n local-storage
    ```

    **Example output:**
    ```
    $ kubectl logs -f diskmaker-discovery-lh5vt -c diskmaker-discovery -n local-storage
    I1028 09:34:14.613039   50525 diskmaker.go:21] Go Version: go1.18.4
    I1028 09:34:14.613156   50525 diskmaker.go:22] Go OS/Arch: linux/amd64
    I1028 09:34:14.613161   50525 diskmaker.go:23] local-storage-diskmaker Version: v4.12.0-202210061001.p0.g7670056.assembly.stream-0-g1a3cbf8-dirty
    I1028 09:34:14.613314   50525 config.go:74] Port: 8383
    I1028 09:34:15.666103   50525 request.go:601] Waited for 1.034907935s due to client-side throttling, not priority and fairness, request: GET:https://172.21.0.1:443/apis/project.openshift.io/v1?timeout=32s
    I1028 09:34:17.233573   50525 discovery.go:67] starting device discovery
    I1028 09:34:17.304151   50525 sync_result.go:68] successfully created LocalVolumeDiscoveryResult resource
    I1028 09:34:17.304224   50525 event.go:285] Event(v1.ObjectReference{Kind:"LocalVolumeDiscovery", Namespace:"local-storage", Name:"auto-discover-devices", UID:"4e7a35a5-70c7-4b59-a8b4-1af24a482407", APIVersion:"local.storage.openshift.io/v1alpha1", ResourceVersion:"5359296", FieldPath:""}): type: 'Normal' reason: 'CreatedDiscoveryResultObject' bha-lv-06 - successfully created LocalVolumeDiscoveryResult resource
    I1028 09:34:17.344471   50525 discovery.go:211] ignoring root device "vda"
    I1028 09:34:17.344628   50525 discovery.go:121] valid block devices: [{Name:vda1 KName:vda1 Type:part Model: Vendor: State: FSType: Size:1048576 Rotational:1 ReadOnly:0 Removable:0 PathByID: Serial: PartLabel:} {Name:vda2 KName:vda2 Type:part Model: Vendor: State: FSType:vfat Size:104857600 Rotational:1 ReadOnly:0 Removable:0 PathByID: Serial: PartLabel:} {Name:vda3 KName:vda3 Type:part Model: Vendor: State: FSType:xfs Size:107267210752 Rotational:1 ReadOnly:0 Removable:0 PathByID: Serial: PartLabel:} {Name:vdb KName:vdb Type:disk Model: Vendor:0x1af4 State: FSType:ext4 Size:150000001024 Rotational:1 ReadOnly:0 Removable:0 PathByID: Serial:3f49e9c1-e0a4-4771-8 PartLabel:} {Name:vdc KName:vdc Type:disk Model: Vendor:0x1af4 State: FSType:iso9660 Size:391168 Rotational:1 ReadOnly:0 Removable:0 PathByID: Serial:cloud-init-0737_3e3e PartLabel:} {Name:vdd KName:vdd Type:disk Model: Vendor:0x1af4 State: FSType:swap Size:45056 Rotational:1 ReadOnly:0 Removable:0 PathByID: Serial:cloud-init- PartLabel:}]
    I1028 12:09:18.685429   50525 discovery.go:271] device "vda1" is available
    I1028 12:09:18.685547   50525 discovery.go:232] device "vda2" with filesystem "vfat" is not available
    I1028 12:09:18.685640   50525 discovery.go:232] device "vda3" with filesystem "xfs" is not available
    I1028 12:09:18.685745   50525 discovery.go:232] device "vdb" with filesystem "ext4" is not available
    I1028 12:09:18.685887   50525 discovery.go:232] device "vdc" with filesystem "iso9660" is not available
    I1028 12:09:18.686012   50525 discovery.go:232] device "vdd" with filesystem "swap" is not available
    I1028 12:09:18.688866   50525 discovery.go:271] device "vde" is available
    I1028 12:09:18.688888   50525 discovery.go:124] discovered devices: [{DeviceID:/dev/disk/by-id/virtio-0737-aaeaa6a5-6bfc-4-part1 Path:/dev/vda1 Model: Type:part Vendor: Serial: Size:1048576 Property:Rotational FSType: Status:{State:Available}} {DeviceID:/dev/disk/by-id/virtio-0737-aaeaa6a5-6bfc-4-part2 Path:/dev/vda2 Model: Type:part Vendor: Serial: Size:104857600 Property:Rotational FSType:vfat Status:{State:NotAvailable}} {DeviceID:/dev/disk/by-id/virtio-0737-aaeaa6a5-6bfc-4-part3 Path:/dev/vda3 Model: Type:part Vendor: Serial: Size:107267210752 Property:Rotational FSType:xfs Status:{State:NotAvailable}} {DeviceID:/dev/disk/by-id/virtio-3f49e9c1-e0a4-4771-8 Path:/dev/vdb Model: Type:disk Vendor:0x1af4 Serial:3f49e9c1-e0a4-4771-8 Size:150000001024 Property:Rotational FSType:ext4 Status:{State:NotAvailable}} {DeviceID:/dev/disk/by-id/virtio-cloud-init-0737_3e3e Path:/dev/vdc Model: Type:disk Vendor:0x1af4 Serial:cloud-init-0737_3e3e Size:391168 Property:Rotational FSType:iso9660 Status:{State:NotAvailable}} {DeviceID:/dev/disk/by-id/virtio-cloud-init- Path:/dev/vdd Model: Type:disk Vendor:0x1af4 Serial:cloud-init- Size:45056 Property:Rotational FSType:swap Status:{State:NotAvailable}} {DeviceID:/dev/disk/by-id/virtio-0737-9d81760d-51ec-4 Path:/dev/vde Model: Type:disk Vendor:0x1af4 Serial:0737-9d81760d-51ec-4 Size:10737418240 Property:Rotational FSType: Status:{State:Available}}]
    ```

11. Verify the PV is created.
    ```sh
    oc get pv
    ```
    
    
## References
   - https://docs.openshift.com/container-platform/4.12/storage/persistent_storage/persistent-storage-local.html
