## How to use the template to deploy aws-ebs-csi-driver on a Satellite Cluster?

### Features supported
- Dyanamic Provisioning
- Volume Resizing
- Volume Snapshot 

### Pre-reqs
- The driver requires IAM permission to talk to Amazon EBS to manage the volume on user's behalf. Create an IAM user with proper permission and get *Access Key* and *Secret Key*.
- You need to provide IOPS per GB, required for *io2* volume type. Refer [EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html) for more details.


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
     $ ibmcloud sat storage template get --name aws-ebs-csi-driver --version 0.8.2
     Getting Satellite storage template details...
     OK
                        
     Name:           aws-ebs-csi-driver   
     Display Name:   AWS EBS CSI driver   
     Version:        0.8.2   

     Custom Parameters
     Name                   Display Name           Description            Required  Type   Default   
     aws-access-key         AWS Access Key         AWS Access Key         true      -      -
     aws-secret-access-key  AWS Secret Access Key  AWS Secret Access Key  true      -      -
     ```
   - Create Storage Configuration
     ```
     $ ibmcloud sat storage config create --name aws-ebs-conf --template-name aws-ebs-csi-driver --template-version 0.8.2 -p aws-access-key=<access-key-without-base64-encoding> -p aws-secret-access-key=<secret-access-key-without-base64-encoding>
     Creating Satellite storage configuration...
     OK
     Storage configuration 'aws-ebs-conf' was successfully created with ID '870b9108-859d-4e6b-b24d-f4380bad9649'.
     ```
     **Note:** 
7. Deploy the configuration
   - Create assignment
     ```
     $ ibmcloud sat storage assignment create --name install-ebs --cluster-group aws-cluster-group --configuration aws-ebs-conf
     Creating assignment...
     OK
     Assignment install-ebs was successfully created with ID 2c35a018-fb22-4d33-98ce-16c42179cab7.
     ```

8. Check the K8S resources on the cluster
   ```
   $ oc get sc | grep aws-block                                                              
     sat-aws-block-bronze      ebs.csi.aws.com    Delete          WaitForFirstConsumer   true                   9h
     sat-aws-block-gold        ebs.csi.aws.com    Delete          WaitForFirstConsumer   true                   9h
     sat-aws-block-silver      ebs.csi.aws.com    Delete          WaitForFirstConsumer   true                   9h


   $ oc get pods -n kube-system | grep ebs                                                   
     ebs-csi-controller-5dc5468b9-b7t4f      6/6     Running   0          9h
     ebs-csi-controller-5dc5468b9-t9tqr      6/6     Running   0          9h
     ebs-csi-node-5gl6t                      3/3     Running   0          9h
     ebs-csi-node-7dx64                      3/3     Running   0          9h
     ebs-csi-node-dt26z                      3/3     Running   0          9h
     ebs-snapshot-controller-0               1/1     Running   0          9h
   ```

### Verification   
- Follow [link](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/dynamic-provisioning) to create PVC and mount PVC inside pod.

### References
- AWS-EBS-CSI-Driver: https://github.com/kubernetes-sigs/aws-ebs-csi-driver
- Examples: https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes
- Amazon EBS: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html