# Deploy Local Volume Block on a Satellite Cluster
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
     $ oc label nodes 52.117.85.9 52.117.85.12 "storage=localvol"
     node/52.117.85.9 labeled
     node/52.117.85.12 labeled
     ```

4. Login using IBM Cloud CLI
   - Login 
     ``` 
     $ ibmcloud login --sso -a https://cloud.ibm.com -r us-east -g default -u <IBMid>
     ```

5. Create Cluster Group
   - From IBM Cloud Web Console
     > https://cloud.ibm.com/satellite/clusters -> Cluster groups -> Create cluster group
   - Add cluster to the group
     > https://cloud.ibm.com/satellite/groups -> select the cluster group -> Clusters -> Add clusters


## Local Volume parameters & how to retrieve them

Parameter | Required? | Description | Default value if not provided | 
--- | --- | --- | --- |
label-key | Required | You can retrieve this parameter by using ```$ oc get nodes --show-labels``` command | N/A | 
Label-value | Required | You can retrieve this parameter by using ```$ oc get nodes --show-labels``` command | N/A | 
devicepath | Required |You can retrieve this parameter by using following commands ```$ oc get nodes``` ```$ oc debug node/<node name> ``` ```# chroot /host``` ```# lsblk``` | N/A | 

## Default storage classes

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| `sat-local-block-gold ` | Block | N/A | N/A | N/A | N/A | Retain | 


## Creating the Local Volume Block storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name localvol-block-config --template-name local-volume-block --template-version 4.5 -p "label-key=storage" -p "label-value=localvol" -p "devicepath=/dev/xvdc"
```
## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name localvol-block-assign --group satClusterGrp --config localvol-block-config
```

## Verifying your Local Volume Block storage configuration is assigned to your clusters
   ```
   $ oc get all -n local-storage
   
   NAME                                         READY   STATUS    RESTARTS   AGE
   pod/local-disk-local-diskmaker-qdrjs         1/1     Running   0          100s
   pod/local-disk-local-provisioner-b6v4n       1/1     Running   0          100s
   pod/local-storage-operator-96c444dfc-m25g7   1/1     Running   0          104s

   NAME                             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
   service/local-storage-operator   ClusterIP   172.21.92.28   <none>        60000/TCP   101s

   NAME                                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
   daemonset.apps/local-disk-local-diskmaker     1         1         1       1            1           <none>          101s
   daemonset.apps/local-disk-local-provisioner   1         1         1       1            1           <none>          101s

   NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/local-storage-operator   1/1     1            1           106s

   NAME                                               DESIRED   CURRENT   READY   AGE
   replicaset.apps/local-storage-operator-96c444dfc   1         1         1       106s

   ```
   ```
   $ oc get pv
   NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS           REASON   AGE
   local-pv-88842685   50Gi       RWO            Delete           Available           sat-local-block-gold            90s

   ```
**Example output**
![Image of output](https://github.com/Hemlata3011/ibm-satellite-storage/blob/master/config-templates/redhat/local-volume-block/4.5/localVolBlk.png)

**Follow** the [link](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-local.html) to create the persistent volume claim and attach the claim to a pod.

## Troubleshooting

**Local PV creation fails**
   - Check for razeedeploy
     ```
     oc get ns
     ```
     ```
     oc get pods -n razeedeploy
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
     
   6. Get the logs for the `local-disk-local-diskmaker` pod.
     ```sh
     kubectl logs -f pod/local-disk-local-diskmaker-7ww2j -n local-storage
     ```
     
     **Example output:**
     ```
     I0213 06:19:35.103830       1 diskmaker.go:24] Go Version: go1.13.15
     I0213 06:19:35.104141       1 diskmaker.go:25] Go OS/Arch: linux/amd64
     I0213 06:19:35.104148       1 diskmaker.go:26] local-storage-diskmaker Version: v4.5.0-202101300210.p0-0-ged6884f-dirty
     E0213 06:19:40.697628       1 diskmaker.go:203] failed to acquire lock on device /dev/xvde
     E0213 06:19:40.697657       1 diskmaker.go:180] error symlinking /dev/xvdc to /mnt/local-storage/sat-local-block-gold/xvdc: error acquiring exclusive lock on /dev/xvdc
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
     rm -rf  /mnt/local-storage/sat-local-block-gold/xvdc
     ```

2.  You can check the local-disk-local-provisioner pod logs to verify if the pv got created with specified volume
    ```
    $ kubectl logs -f pod/local-disk-local-provisioner-xstjh -n local-storage 
    ...
    I0213 06:30:44.146148       1 common.go:382] Creating client using in-cluster config
    I0213 06:30:44.174765       1 main.go:85] Starting controller
    I0213 06:30:44.174800       1 main.go:100] Starting metrics server at :8080
    I0213 06:30:44.175265       1 controller.go:45] Initializing volume cache
    I0213 06:30:44.378164       1 controller.go:108] Controller started
    I0213 06:30:44.378851       1 discovery.go:304] Found new volume at host path "/mnt/local-storage/sat-local-bolck-gold/xvdc" with capacity 53687091200, creating Local PV "local-pv-1d14680"
    I0213 06:30:44.396328       1 cache.go:55] Added pv "local-pv-1d14680" to cache
    I0213 06:30:44.396471       1 discovery.go:337] Created PV "local-pv-1d14680" for volume at "/mnt/local-storage/sat-local-block-gold/xvdc"
    I0213 06:30:44.462244       1 cache.go:64] Updated pv "local-pv-1d14680" to cache
    ```
    
## References
   - https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-local.html
