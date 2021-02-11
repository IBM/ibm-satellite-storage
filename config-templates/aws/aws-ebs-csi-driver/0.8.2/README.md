# AWS EBS CSI Driver

AWS EBS CSI driver implements the CSI specification for container orchestrators to manage the lifecycle of Amazon EBS volumes.

**Features supported:**
- Dyanamic Provisioning
- Volume Resizing
- Volume Snapshot 

## Prerequisites
**Planning consideration for Infra Admin**
-  For better IOPS and throughput, use instance built on *Nitro System*. Refer to 
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html

**Planning consideration for Location Admin**
- The driver requires IAM permission to talk to Amazon EBS to manage the volume on user's behalf. Create an IAM user with proper permission and get *AWS Access Keys*. Refer to https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html

- Volume type *io2* requires iopsPerGB, default is set to *10 IOPS/GB*.
  To set different value for iopsPerGB, refer to https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html


## AWS EBS CSI Driver parameters & how to retrieve them

Retrieve all parameters required by this template
```
ibmcloud sat storage template get --name aws-ebs-csi-driver --version 0.8.2
```

**AWS EBS CSI Driver parameters**

| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `aws-access-key` | Required | This is a required parameter. | N/A |
| `aws-secret-access-key` | Required | This is a required parameter. | N/A | |
| `iops-per-gb` | Optional | This is an optional parameter.  | 10 |


## Default storage classes

| Storage class name | EBS Volume Type | FileSystem Type | default IOPS per GB | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- | --- |
| `sat-aws-block-gold` | io2 | ext4 | 10 | 4 GiB - 16 TiB | SSD | Delete | 
| `sat-aws-block-silver` | gp3 | ext4 | N/A | 1 GiB - 16 TiB | SSD | Delete | 
| `sat-aws-block-bronze` | st1 | ext4 | N/A | 125 GiB - 16 TiB | HDD | Delete | 


## Creating the AWS EBS CSI Driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name aws-ebs-conf --template-name aws-ebs-csi-driver --template-version 0.8.2 -p "aws-access-key=<access-key-without-base64-encoding>" -p "aws-secret-access-key=<secret-access-key-without-base64-encoding>" -p "iops-per-gb=<iops-per-gb>"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name install-ebs --cluster-group <cluster-group> --configuration aws-ebs-conf
```

## Verifying your AWS EBS CSI Driver storage configuration is assigned to your clusters

Provide steps to retrieve any driver pods, storage classes, or any other resources that are deployed with your storage configuration.

```
$ oc get pods -n kube-system | grep ebs    
ebs-csi-controller-5dc5468b9-d8c5c      6/6     Running   0          2d8h
ebs-csi-controller-5dc5468b9-p2tw4      6/6     Running   0          2d8h
ebs-csi-node-2vt9v                      3/3     Running   0          2d8h
ebs-csi-node-7wccn                      3/3     Running   0          2d8h
ebs-csi-node-rllvw                      3/3     Running   0          2d8h
ebs-snapshot-controller-0               1/1     Running   0          2d8h
```

```
$ oc get sc | grep aws-block               
sat-aws-block-bronze      ebs.csi.aws.com    Delete          WaitForFirstConsumer   true                   2d8h
sat-aws-block-gold        ebs.csi.aws.com    Delete          WaitForFirstConsumer   true                   2d8h
sat-aws-block-silver      ebs.csi.aws.com    Delete          WaitForFirstConsumer   true                   2d8h
```


**Example output**

![Example Output](./images/output.png)


## Troubleshooting


## Reference

- AWS-EBS-CSI-Driver: https://github.com/kubernetes-sigs/aws-ebs-csi-driver
- Examples: https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes
- Amazon EBS: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html
- AWS Support: https://console.aws.amazon.com/support/home#/