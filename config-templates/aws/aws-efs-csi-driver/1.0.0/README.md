## How to use the template to deploy aws-efs-csi-driver on a Satellite Cluster?

### Pre-reqs 
- Currently only *static provisioning* is supported. This means an [AWS EFS file system](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) needs to be created manually on AWS first. After that it can be mounted inside a container as a volume using the driver.


### Detailed steps
1. Login into the Cluster using `oc CLI`
   - Login to [IBM Cloud Web Console](https://cloud.ibm.com/)
	 - Go to [Clusters page](https://cloud.ibm.com/satellite/clusters)
   - Click on the row with the cluster details, You will get your cluster page
   - Click on `Manage Cluster` and in next page click on `OpenShift web console`
   - In OpenShift console click on your user ID at top right corner and then click on `Copy Login Command`
   - In next page click on `Display Token`
   - Under `Log in with this token` the `oc login --token=XXXX ...` will be displayed, copy the command and execute on your local system

2. Verify that all the worker nodes are healthy. 
   ```
    $ oc get nodes
    172.33.27.167   Ready    master,worker   3d17h   v1.19.0+3b01205
    172.33.32.37    Ready    master,worker   3d17h   v1.19.0+3b01205
    172.33.9.160    Ready    master,worker   3d17h   v1.19.0+3b01205
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
     > https://cloud.ibm.com/satellite/groups -> select the cluster group -> Clusters -> Add cluster

5. View the cluster group from CLI
   ```
    ibmcloud sat cluster-group get --cluster-group aws-cluster-group 
    OK
    Cluster Group            
    Name:                 aws-cluster-group   
    ID:                   c08d6a40-4790-4572-ade6-f85f147c2d9e   
    Created:              2020-10-24T09:55:32.788Z   
    Cluster Count:        1   
    Subscription Count:   0   

    Subscriptions      
    Name   ID   Version  

    Clusters      
    Name                ID                                   Location   
    aws-satellite-prod  c27efdeb-4b04-4a2d-a5dc-84581874a2de c0dvnj9w0odhl72ekhh0
   ```

6. Create storage configuration using existing template
   - Review the required parameters for the template
     ```
     $ ibmcloud sat storage template get --name aws-efs-csi-driver --version 1.0.0
     Getting Satellite storage template details...
     OK
                        
     Name:           aws-efs-csi-driver   
     Display Name:   AWS EFS CSI driver   
     Version:        1.0.0   

     Custom Parameters
     Name    Display Name      Description        Required   Type   Default   
     dummy   Dummy Parameter   Dummmy Parameter   false      -      dummy
     ```
   - Create Storage Configuration
     ```
     $ ibmcloud sat storage config create --name aws-efs-conf --template-name aws-efs-csi-driver --template-version 1.0.0
     Creating Satellite storage configuration...
     OK
     Storage configuration 'aws-efs-conf' was successfully created with ID '870b9108-859d-4e6b-b24d-f4380bad9649'.
     ```
7. Deploy the configuration
   - Create assignment
     ```
     $ ibmcloud sat storage assignment create --name install-efs --cluster-group aws-cluster-group --configuration aws-efs-conf
     Creating assignment...
     OK
     Assignment install-efs was successfully created with ID 2c35a018-fb22-4d33-98ce-16c42179cab7.
     ```

8. Check the K8S resources on the cluster
   ```
   $ oc get sc | grep aws-file 
   sat-aws-file-gold              efs.csi.aws.com    Delete          Immediate              false                  20h


   $ oc get pods -n kube-system | grep efs    
   efs-csi-node-4gkzx                      3/3     Running   0          2d22h
   efs-csi-node-r8g5d                      3/3     Running   0          2d22h
   efs-csi-node-td4wc                      3/3     Running   0          2d22h
   ```

### Verification   
- Follow [link](https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/examples/kubernetes/static_provisioning/README.md) to create PV using EFS volume, PVC and mount PVC inside pod.

### References
- AWS-EFS-CSI-Driver: https://github.com/kubernetes-sigs/aws-efs-csi-driver
- Examples: https://github.com/kubernetes-sigs/aws-efs-csi-driver/tree/master/examples/kubernetes
- Amazon EFS: https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html