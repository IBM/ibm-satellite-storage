This README explains the steps involved in setting up Trident to serve NetApp
storage in IBM Satellite Clusters. In a nutshell:

1. Install Trident on the Satellite Cluster. This step is assumed to be handled
   by IBM Satellite as part of the Satellite cluster set up.
2. Create a Trident backend. This requires information about the ONTAP storage
   cluster being used, such as credentials and management IPs.
3. Create multiple Kubernetes StorageClasses. These storage classes will provide
   differential classes of service for Satellite end users to reference when
   creating PVCs.

### How to use the template to deploy and set up Trident on a Satellite Cluster?

#### Pre-reqs
- Trident must be installed by Satellite as part of cluster set up.
- The backend ONTAP cluster has been configured to be used as a Trident backend.
  Namely:
    - A SVM must be dedicated for Trident. Volumes created by Trident will be placed
      in this SVM.
    - One or more aggregates must be assigned to the SVM.
    - One or more dataLIFs must be created for the SVM. Depending on the protocol
      used (NFS/iSCSI), at least one dataLIF is required.
    - NFS/iSCSI services must be started on the SVM.
    - Ensure the SVM has snapshot policies set up to be used by Trident. *It is
      recommended a `default` snapshot policy is created on the SVM being used*.

### Detailed steps
1. Login into the Cluster using `OC CLI`
   - Login to [IBM Cloud Web Console](https://cloud.ibm.com/)
	 - Go to [Clusters page](https://cloud.ibm.com/satellite/clusters)
   - Click on the row with the cluster details, You will get your cluster page
   - Click on `Manage Cluster` and in next page click on `OpenShift web console`
   - In OpenShift console click on your user ID at top right corner and then click on `Copy Login Command`
   - In next page click on `Display Token`
   - Under `Log in with this token` the `oc login --token=XXXX ...` will be displayed, copy the command and execute on your local system

2. Check to see if Trident is installed in the cluster. Assuming the namespace
   that Trident is installed is `trident`
   ```
     $ oc get pods -n trident
     NAME                                READY   STATUS    RESTARTS   AGE
     trident-csi-79df798bdc-m79dc        6/6     Running   0          1m41s
     trident-csi-xrst8                   2/2     Running   0          1m41s
     trident-csi-yxtn2                   2/2     Running   0          1m41s
     trident-csi-aowni                   2/2     Running   0          1m41s
     trident-operator-5574dbbc68-nthjv   1/1     Running   0          1m52s
     ```
   Note: If Trident is installed using the [Trident Operator](https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/deploying/operator-deploy.html)
   the `trident-operator-xxxxxxxx-xxxxx` pod will be present. If Trident is
   installed with [tridentctl](https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/deploying/tridentctl-deploy.html), the `trident-operator-xxxxxxxx-xxxxx` won't
   be seen.

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
    ibmcloud sat cluster-group get --cluster-group netapp-trident
    OK
    Cluster Group
    Name:                 netapp-trident
    ID:                   ce55ec92-167b-4047-b098-278da83ab369
    Created:              2021-02-02T16:53:59.577Z
    Cluster Count:        1
    Subscription Count:   0

    Subscriptions
    Name   ID   Version

    Clusters
    Name          ID                                     Location
    trident       09db2fd1-8f40-4784-9e67-36b9f6594a9b   bvvec9ew0sbeueqsd2r0
   ```

6. Create storage configuration using existing template
   - Review the required parameters for the template
     ```
     $ ibmcloud sat storage template get --name netapp-trident --version 4.5
     Getting Satellite storage template details...
     OK

     Name:           netapp-trident
     Display Name:   Netapp Trident version 20.07
     Version:        4.5

     Custom Parameters
     Name          Display Name           Description                                Required   Type   Default
     namespace     Namespace              Controller Namespace                       false      -      trident
     storagedriver Storage Driver         Storage Driver                             false      -      ontap-nas
     .
     .
     .
     user-pass     User Password          User Password                              true      -      -
     ```
   - Create Storage Configuration

     ```
     $ ibmcloud sat storage config create --name trident-deploy --template-name netapp-trident --template-version 4.5 -p "storagedriver=ontap-nas" -p "namespace=trident" -p "mgmt-lif=192.168.0.1" -p "data-lif=192.168.0.11" -p "svm-id=nfs_svm" -p "autoExportPolicy=true" -p "autoExportCIDRs=[“10.0.0.0/22"] -p "user-id=vsadmin" -p "user-pass=secret"
     Creating Satellite storage configuration...
     OK
     Storage configuration 'trident-deploy' was successfully created with ID '870b9108-859d-4e6b-b24d-f4380bad9649'.
     ```
7. Deploy the configuration
   - Create assignment
     ```
     $ ibmcloud sat storage assignment create --name trident-vol --cluster-group netapp-trident --configuration trident-deploy
     Creating assignment...
     OK
     Assignment trident-vol was successfully created with ID 2c35a018-fb22-4d33-98ce-16c42179cab7.
     ```

8. Check the K8S resources on the cluster
   ```
   $ oc get all -n trident
   NAME                                    READY   STATUS    RESTARTS   AGE
   pod/trident-csi-68d979fb85-dsrmn        6/6     Running   0          1d
   pod/trident-csi-8jfhf                   2/2     Running   0          1d
   pod/trident-csi-jtnjz                   2/2     Running   0          1d
   pod/trident-csi-lcxvh                   2/2     Running   0          1d
   pod/trident-operator-7569744d4f-hwgnr   1/1     Running   0          1d

   NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
   service/trident-csi   ClusterIP   10.108.174.125   <none>        34571/TCP,9220/TCP   1d

   NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                     AGE
   daemonset.apps/trident-csi   3         3         3       3            3           kubernetes.io/arch=amd64,kubernetes.io/os=linux   1d

   NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/trident-csi        1/1     1            1           1d
   deployment.apps/trident-operator   1/1     1            1           1d

   NAME                                          DESIRED   CURRENT   READY   AGE
   replicaset.apps/trident-csi-68d979fb85        1         1         1       1d
   replicaset.apps/trident-operator-7569744d4f   1         1         1       1d
   ```
