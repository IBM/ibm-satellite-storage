### How to use the template to deploy cos on a Satellite Cluster?

#### Pre-reqs 
- 

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
    ibmcloud sat group get --group ibm-cluster-group 
    OK
    Cluster Group            
    Name:                 ibm-cluster-group   
    ID:                   3f930291-812b-4061-bc2b-c5459d4937f7   
    Created:              2021-02-10T08:14:17.463Z   
    Cluster Count:        1   
    Subscription Count:   1   
    
    Subscriptions      
    Name          ID                                     Version   
    install-cos   eecec6a2-8dfe-40db-81d8-c121b96fa153   cos-config-bha-01   
    
    Clusters      
    Name                    ID                                     Location   
    bha-satellite-cluster   d000adfa-d029-4a34-9058-e1494222a441   c0gv9532004p89jm8h9g     
   ```

6. Create storage configuration using existing template
   - Review the required parameters for the template
     ```
     $ ibmcloud sat storage template get --name local-volume --version 4.5
     Getting Satellite storage template details...
     OK
                   
     Name:           cos   
     Display Name:   Cloud Object Storage   
     Version:        2.0.6   

     Custom Parameters
     Name          Display Name           Description                                Required   Type   Default   
     namespace     Plugin Namespace       Namespace where Plugin has to be deployed  false      -      kube-system   
     license       License                Change license to true to indicate have read and agreed to license agreement  false      -      false   
     cpu           cpu                    cpu                                        false      -      200m
     memory        memory                 memory                                     false      -      128Mi          
     ```
   - Create Storage Configuration
     ```
     $ ibmcloud sat storage config create --name cos-config-bha-01 --template-name cos --template-version 2.0.6 --param license=true
     Creating Satellite storage configuration...
     OK
     Storage configuration 'cos-config-bha-01' was successfully created with ID '9c0bb0d0-bc13-4198-b801-fb5caf066ea8'.
     ```
7. Deploy the configuration
   - Create assignment
     ```
     $ ibmcloud sat storage assignment create --name install-cos --group ibm-cluster-group --config cos-config-bha-01 
     Creating assignment...
     OK
     Assignment install-cos was successfully created with ID eecec6a2-8dfe-40db-81d8-c121b96fa153.
     ```

8. Check the K8S resources on the cluster
   ```
   $ oc get pods -n kube-system | grep object
     ibmcloud-object-storage-driver-75gxg           1/1     Running     0          37m
     ibmcloud-object-storage-driver-wtfmv           1/1     Running     0          37m
     ibmcloud-object-storage-driver-zc7ls           1/1     Running     0          37m
     ibmcloud-object-storage-plugin-9859fb9-4tc8p   1/1     Running     0          37m
     
   $ oc get pods -n kube-system | grep cos
     cos-plugin-installer-pod                       0/1     Completed   0          44m
   
   $ helm ls -A | grep object
     ibm-object-storage-plugin	kube-system	1       	2021-02-11 07:34:52.048136127 +0000 UTC	deployed	ibm-object-storage-plugin-2.0.6	2.0.6 
   ```

### Verification   
- Follow [link](https://cloud.ibm.com/docs/containers?topic=containers-object_storage#add_cos) to create PVC and mount PVC inside pod.

### References
- Cloud Object Storage: https://cloud.ibm.com/docs/containers?topic=containers-object_storage


