### How to use the template to deploy local-volume on a Satellite Cluster?

#### Pre-reqs 
- The cluster worker nodes (hosts) should have an additional free disk.

### Detailed steps
1. Login into the Cluster using `OC CLI`
   - Login to [IBM Cloud Web Console](https://cloud.ibm.com/)
	 - Go to [Clusters page](https://cloud.ibm.com/satellite/clusters)
   - Click on the row with the cluster details, You will get your cluster page
   - Click on `Manage Cluster` and in next page click on `OpenShift web console`
   - In OpenShift console click on your user ID at top right corner and then click on `Copy Login Command`
   - In next page click on `Display Token`
   - Under `Log in with this token` the `oc login --token=XXXX ...` will be displayed, copy the command and execute on your local system

2. Add label to the worker nodes, one with additional disk
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

3. Login using IBM Cloud CLI
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

4. Create Cluster Group
   - From IBM Cloud Web Console
     > https://cloud.ibm.com/satellite/clusters -> Cluster groups -> Create cluster group
   - Add cluster to the group
     > https://cloud.ibm.com/satellite/groups -> select the cluster group -> Clusters -> Add clusters

5. View the cluster group from CLI
   ```
    ibmcloud sat cluster-group get --cluster-group localvol 
    OK
    Cluster Group            
    Name:                 localvol   
    ID:                   ce55ec92-167b-4047-b098-278da83ab369   
    Created:              2021-02-02T16:53:59.577Z   
    Cluster Count:        1   
    Subscription Count:   0   

    Subscriptions      
    Name   ID   Version   

    Clusters      
    Name          ID                                     Location   
    satlocalvol   09db2fd1-8f40-4784-9e67-36b9f6594a9b   bvvec9ew0sbeueqsd2r0   
   ```

6. Create storage configuration using existing template
   - Review the required parameters for the template
     ```
     $ ibmcloud sat storage template get --name local-volume --version 4.5
     Getting Satellite storage template details...
     OK
                   
     Name:           local-volume   
     Display Name:   Persistent storage using local volumes   
     Version:        4.5   

     Custom Parameters
     Name          Display Name           Description                                Required   Type   Default   
     devicepath    Device Path            Local storage device path, e.g. /dev/sdc   false      -      -   
     label-key     Node Label Key         Node label key                             false      -      -   
     label-value   Node Label Key Value   Node label key value                       false      -      -   
     ```
   - Create Storage Configuration
     ```
     $ ibmcloud sat storage config create --name localvol-conf --template-name local-volume --template-version 4.5 -p "label-key=storage" -p "label-value=localvol" -p "devicepath=/dev/xvde"
     Creating Satellite storage configuration...
     OK
     Storage configuration 'localvol-conf' was successfully created with ID '870b9108-859d-4e6b-b24d-f4380bad9649'.
     ```
7. Deploy the configuration
   - Create assignment
     ```
     $ ibmcloud sat storage assignment create --name local-vol --cluster-group localvol --configuration localvol-conf
     Creating assignment...
     OK
     Assignment local-vol was successfully created with ID 2c35a018-fb22-4d33-98ce-16c42179cab7.
     ```

8. Check the K8S resources on the cluster
   ```
   $ oc get all -n local-storage
   NAME                                          READY   STATUS    RESTARTS   AGE
   pod/local-disk-local-diskmaker-mgcjw          1/1     Running   0          2m25s
   pod/local-disk-local-diskmaker-rtsbq          1/1     Running   0          2m25s
   pod/local-disk-local-provisioner-67pbt        1/1     Running   0          2m25s
   pod/local-disk-local-provisioner-l9h6h        1/1     Running   0          2m25s
   pod/local-storage-operator-7c948cf678-bfvrs   1/1     Running   0          2m29s

   NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
   service/local-storage-operator   ClusterIP   172.21.219.255   <none>        60000/TCP   2m27s

   NAME                                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
   daemonset.apps/local-disk-local-diskmaker     2         2         2       2            2           <none>          2m26s
   daemonset.apps/local-disk-local-provisioner   2         2         2       2            2           <none>          2m26s

   NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/local-storage-operator   1/1     1            1           2m31s

   NAME                                                DESIRED   CURRENT   READY   AGE
   replicaset.apps/local-storage-operator-7c948cf678   1         1         1       2m31s
   ```
   ```
   $ oc get pv
   NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
   local-pv-8a72fc8d   50Gi       RWO            Delete           Available           localblock-sc            47m
   local-pv-8c400223   50Gi       RWO            Delete           Available           localblock-sc            47m
   ```
