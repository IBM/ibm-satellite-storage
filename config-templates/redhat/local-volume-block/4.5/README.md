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
ibmcloud sat storage config create --name localvol-block-config --template-name local-volume-block --template-version 4.5 -p "label-key=stoarge" -p "label-value=localvol" -p "devicepath=/dev/xvdc"
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

## Troubleshooting

1. PVs are not getting created
   - Check for razeedeploy
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
   
