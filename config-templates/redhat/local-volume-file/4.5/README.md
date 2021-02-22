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

5. Create Cluster Group
   - From IBM Cloud Web Console
     > https://cloud.ibm.com/satellite/clusters -> Cluster groups -> Create cluster group
   - Add cluster to the group
     > https://cloud.ibm.com/satellite/groups -> select the cluster group -> Clusters -> Add clusters


## Local Volume parameters & how to retrieve them

Parameter | Required? | Description | Default value if not provided | 
----------| ----------| ------------| ----------------------------- |
label-key | Required  | You can retrieve this parameter by using ```$ oc get nodes --show-labels``` command | N/A | 
label-value| Required | You can retrieve this parameter by using ```$ oc get nodes --show-labels``` command | N/A | 
devicepath | Required | You can retrieve this parameter by using following commands ```$ oc get nodes``` ```$ oc debug node/<node name> ``` ```# chroot /host``` ```# lsblk``` | N/A | 
fstype     | Required | File System type of your choice     |          ext4|


## Default storage classes

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| ------------------ | ---- | ----------- | ---- | ---------- | ----------| -------------- |
| sat-local-file-gold| File | ext4/xfs    | N/A  | N/A        | N/A       | Retain         | 


## Creating the Local Volume File storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name localvol-file-config --template-name local-volume-file --template-version 4.5 -p "label-key=storage" -p "label-value=localfile" -p "devicepath=/dev/xvde"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name localvol-file-assign --group satClusterGrp --config localvol-file-config
```

## Verifying your Local Volume File storage configuration is assigned to your clusters

```
$ oc get all -n local-storage
NAME                                         READY   STATUS    RESTARTS   AGE
pod/local-disk-local-diskmaker-cpk4r         1/1     Running   0          30s
pod/local-disk-local-provisioner-xstjh       1/1     Running   0          30s
pod/local-storage-operator-96c444dfc-ttpmq   1/1     Running   0          35s

NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/local-storage-operator   ClusterIP   172.21.173.238   <none>        60000/TCP   32s

NAME                                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/local-disk-local-diskmaker     1         1         1       1            1           <none>          31s
daemonset.apps/local-disk-local-provisioner   1         1         1       1            1           <none>          31s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/local-storage-operator   1/1     1            1           36s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/local-storage-operator-96c444dfc   1         1         1       37s

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
   
**Follow** the [link](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-local.html) to create the persistent volume claim and attach the claim to a pod.
   
   
## Troubleshooting

1.  If the PV is not created, 
    - Check if all resources are created in razeedeploy
       ```
       oc get ns
       ```
       ```
       oc get pods -n razeedeploy
       ```
      - Go to the IBM cloud web console.
        Navigation Menu -> Satellite -> Clusters
      - Select the cluster and click on the overflow menu of that particular cluster.
      - Click on the Re-attach cluster and copy the displayed command and execute it.
    - Check the namespace in which template has been deployed is active
        ```
        $ oc get ns | grep local
        local-storage                                      Active   101m
        ```
    - verify the logs for local-disk-local-diskmaker pod.
    If the logs show as below - 
    ```
    kubectl logs -f pod/local-disk-local-diskmaker-7ww2j -n local-storage01
    I0213 06:19:35.103830       1 diskmaker.go:24] Go Version: go1.13.15
    I0213 06:19:35.104141       1 diskmaker.go:25] Go OS/Arch: linux/amd64
    I0213 06:19:35.104148       1 diskmaker.go:26] local-storage-diskmaker Version: v4.5.0-202101300210.p0-0-ged6884f-dirty
    E0213 06:19:40.697628       1 diskmaker.go:203] failed to acquire lock on device /dev/xvde
    E0213 06:19:40.697657       1 diskmaker.go:180] error symlinking /dev/xvde to /mnt/local-storage/sat-local-file-gold/xvde: error acquiring exclusive lock on /dev/xvde
    ```
    delete the symlink from the node
    ```
    oc debug node/<node-name>
    chroot /host
    rm -rf </path/to/symlink/as/shown/in/logs>
    ```
    Example `rm -rf  /mnt/local-storage/sat-local-file-gold/xvde`

2.  You can check the local-disk-local-provisioner pod logs to verify if the pv got created with specified volume
    ```
    $ kubectl logs -f pod/local-disk-local-provisioner-xstjh -n local-storage
    I0213 06:30:44.145331       1 common.go:320] StorageClass "sat-local-file-gold" configured with MountDir "/mnt/local-storage/sat-local-file-gold", HostDir "/mnt/local-storage/sat-local-file-gold", VolumeMode "Filesystem", FsType "ext4", BlockCleanerCommand ["/scripts/quick_reset.sh"]
    I0213 06:30:44.145494       1 main.go:63] Loaded configuration: {StorageClassConfig:map[sat-local-file-gold:{HostDir:/mnt/local-storage/sat-local-file-gold MountDir:/mnt/local-storage/sat-local-file-gold BlockCleanerCommand:[/scripts/quick_reset.sh] VolumeMode:Filesystem FsType:ext4}] NodeLabelsForPV:[] UseAlphaAPI:false UseJobForCleaning:false MinResyncPeriod:{Duration:5m0s} UseNodeNameOnly:false LabelsForPV:map[storage.openshift.com/local-volume-owner-name:local-disk storage.openshift.com/local-volume-owner-namespace:local-storage]}
    I0213 06:30:44.145538       1 main.go:64] Ready to run...
    W0213 06:30:44.145548       1 main.go:73] MY_NAMESPACE environment variable not set, will be set to default.
    W0213 06:30:44.145562       1 main.go:79] JOB_CONTAINER_IMAGE environment variable not set.
    I0213 06:30:44.146148       1 common.go:382] Creating client using in-cluster config
    I0213 06:30:44.174765       1 main.go:85] Starting controller
    I0213 06:30:44.174800       1 main.go:100] Starting metrics server at :8080
    I0213 06:30:44.175265       1 controller.go:45] Initializing volume cache
    I0213 06:30:44.378164       1 controller.go:108] Controller started
    I0213 06:30:44.378851       1 discovery.go:304] Found new volume at host path "/mnt/local-storage/sat-local-file-gold/xvde" with capacity 53687091200, creating Local PV "local-pv-1d14680", required volumeMode "Filesystem"
    I0213 06:30:44.396328       1 cache.go:55] Added pv "local-pv-1d14680" to cache
    I0213 06:30:44.396471       1 discovery.go:337] Created PV "local-pv-1d14680" for volume at "/mnt/local-storage/sat-local-file-gold/xvde"
    I0213 06:30:44.462244       1 cache.go:64] Updated pv "local-pv-1d14680" to cache
    ```
    
## References
   - https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-local.html
