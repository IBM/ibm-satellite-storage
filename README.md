# ibm-satellite-storage
This repository is used to register the storage for IBM satellite. The vendor can test and create the storage template in the specified format. Once the approval cycle is complete, the storage will be available for IBM satellite offering.
## Directory structure
The following directory structure must be used to register a new driver.
```
storage-templates
---registerd-storage-list.json
---provider-name
   ---storage-name
      ---storage-version
          ---deployment-template.yaml # Contains the kube artifacts for the storage including Deployment,StatefullSet,DaemonSet,Configmap, Secrets,Storage classes
          ---deployment-parameters.json #[Optional] Allows user to can override the values for the parameters while deploying storage to satellite cluster
          ---storage-class-template.yaml #[Optional] Allows user to add new storage classes while deploying storage to satellite cluster
          ---storage-class-parameters.json #[Optionl] Allows user to add new storage classes while deploying storage to satellite cluster
```
1. `storage-list.json`:- This file is common for all of the provider and their storage. It maintains the status of each storage e.g "Testing" , "Unpublished", "Published"
1. `provider-name` :- The name of the provider e.g IBM, AWS, Azure, Netapp, Dell/EMC
1. `storage-name` :- The storage name . The one provider can have multiple storage offerings for IBM satallite
1. `storage-version` :- The vension of the storage. The one storage can have multiple versions
1. `deployment-template.yaml` :- Contains the kube artifacts for the storage including Deployment,StatefullSet,DaemonSet,Configmap, Secrets,Storage classes
1. `deployment-parameters.json`:- This file containts the list of parameters available for user to override the values for the parameters while deploying storage to satellite cluster
1. `storage-class-template.yaml`:- This file contains the storage class template. It allows user to add new storage classes while deploying storage to satellite cluster
1. `storage-class-parameters.json` #[Optionl] This file containts the list of parameters available for user to override the values for the storage parameters while deploying storage to satellite cluster
### Example


```
  storage-templates
	   storage-list.json
     ---aws
        ---aws-ebs-csi-driver
            ---1.0.0
                ---customer-paramerters.json
                ---deployment.yaml
                ---storage-class-template.yaml
                ---stprage-class-parameters.json
     				---1.1.0
                ---customer-paramerters.json
                ---deployment.yaml
                ---storage-class-template.yaml
                ---stprage-class-parameters.json
    ---azure            
        ---azuredisk-csi-driver
     				---1.0.0
              ---customer-paramerters.json
              ---deployment.yaml
              ---storage-class-template.yaml
              ---stprage-class-parameters.json
     				---1.1.0
              ---customer-paramerters.json
              ---deployment.yaml
              ---storage-class-template.yaml
              ---stprage-class-parameters.json
    ---netapp            
        ---trident
     				---27.01
              ---customer-paramerters.json
              ---deployment.yaml
              ---storage-class-template.yaml
              ---stprage-class-parameters.json

```

#### Sample `storage-list.json`
```
[
  {
    "name":ocs",
    "displayname": "OpenShift Container Storage",
    "Description": "OpenShift Container Storage for satallite cluster",
    "default-version": "1.1.0",
    "enabled": "true"
  },
	{
    "name":aws-ebs-csi-driver",
    "displayname": "AWS block CSI driver",
    "Description": "AWS block CSI driver for satallite cluster",
    "default-version": "1.0.0",
    "enabled": "true"
  },
	{
    "name":azuredisk-csi-driver",
    "displayname": "Azure block CSI driver",
    "Description": "Azure block CSI driver for satallite cluster",
    "default-version": "1.1.0",
    "enabled": "true"
  }
	]
```
#### Sample `customer-paramerters.json`
 We try to leverage schematic or catalog service custom parameter code to render the parameters on the UI
```
[
  {
    "description": Cluster Name of OCS",
    "displayname": "Cluster Name",
    "name": "clustername",
    "required": true,
    "type": "text"
  },
  {
    "description": Enter the OSD device path. Mandatory",
    "displayname": "Device Path",
    "name": "devicepath",
    "default" :"/dev/sdc",
    "required": true,
    "type": "text"
  },
  {
    "description": Enter the MON device path. Mandatory",
    "displayname": "File Device Path",
    "name": "filedevicepath ",
    "default" :"/dev/dm-01",
    "required": true,
    "type": "text"
  },
  {
    "description": hardware",
    "displayname": "Hardware",
    "name": "hardware ",
    "required": true,
    "default" :"dedicated",
    "type": "text"
  },
  {
    "description": Secrete ",
    "displayname": "Secrete",
    "name": "secrete",
    "options": [
      {
        "displayname": "Instance IAM profile",
        "value": "ins_iam_profile"
      },
      {
        "displayname": "Secret Config",
        "value": "secret_config"
      }
    ],
    "required": true,
    "type": "dropdown",
    "value": "ins_iam_profile"
  },
  {

    "description": "IBM cloud API key",
    "displayname": "IBM cloud API key",
    "name": "cloudapikey",
    "required": true,
    "type": "password"
    "associations": {
      "parameters": [
        {
          "description": "",
          "displayname": "Secrete",
          "name": "secrete",
          "optionsRefresh": true,
          "showFor": [
            "secret_config"
          ]
        }
      ]
    }
]
```

#### Sample `deployment.yaml`.(OCS)
```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
  name: openshift-storage
spec: {}
---
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
  name: local-storage
spec: {}
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-storage-operatorgroup
  namespace: openshift-storage
spec:
  targetNamespaces:
  - openshift-storage
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: local-storage-operatorgroup
  namespace: local-storage
spec:
  targetNamespaces:
  - local-storage
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ocs-subscription
  namespace: openshift-storage
spec:
    channel: stable-4.4
    name: ocs-operator
    source: redhat-operators
    sourceNamespace: openshift-marketplace
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: local-storage-operator
  namespace: local-storage
spec:
  channel: "4.4"
  name: local-storage-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
---
apiVersion: local.storage.openshift.io/v1
kind: LocalVolume
metadata:
  name: local-block
  namespace: local-storage
spec:
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
        - key: cluster.ocs.openshift.io/openshift-storage
          operator: In
          values:
          - ""
  storageClassDevices:
    - storageClassName: localblock
      volumeMode: Block
      devicePaths:
        - {{ devicepath }}
---
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: {{ cluster-name }}
  namespace: openshift-storage
spec:
  manageNodes: false
  monPVCTemplate:
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: {{ mon-storage-size }} # 20Gi
      storageClassName: {{ mon-storage-class }} # ibmc-block-gold
      volumeMode: Filesystem
  storageDeviceSets:
  - count: 1
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: {{ osd-storage-size }} #1
        storageClassName: {{ osd-storage-class }} #locakblock
        volumeMode: Block
    name: ocs-deviceset
    placement: {}
    portable: false
    replica: 3
    resources: {}


```
#### Sample `storage-class-template.yaml`.(Netapp)
The storage class template is optional . It is required when vendor would like to give ability to user  create the additional storage classes from the given template.  The predefined storage classes can be specified in the `deployment.yaml`. They will be available to user by default.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: {{name}} #ontap-nas-silver
	annotations:
	  storageclass.kubernetes.io/is-default-class={{is_default}} #true/false
provisioner: csi.trident.netapp.io
parameters:
  backendType: {{backendType}} #"ontap-nas"
  snapshots: {{snapshots}} #"false"
  provisioningType: {{provisioningType}} #"thin"
  encryption: {{encryption}} #"false"

```
#### Sample `storage-class-config-parameters.json` . (NetApp)
The config parameters available to create the additional  storage classes
```

[
  {
    "description": Storage class Name to be created",
    "displayname": "Storage Class Name",
    "name": "name",
    "required": true,
    "type": "text"
  },
  {
    "description": Is the storage class default",
    "displayname": "Default?",
    "name": "is_default",
    "default" :"false",
    "required": false,
    "type": "boolen"
  },
  {
    "description": Backend Type",
    "displayname": "Backend Type",
    "name": "backendType ",
    "default" :"ontap-nas",
    "required": false,
    "type": "option"
  },
  {
    "description": Weather to take Snapshots",
    "displayname": "Snapshots",
    "name": "snapshots ",
    "required": false,
    "default" :"false",
    "type": "boolean"
  },
  {
    "description": Enable volume encryption",
    "displayname": "Volume Encryption",
    "name": "encryption ",
    "required": false,
    "default" :"false",
    "type": "boolean"
  },
  {
    "description": Provisioning Type",
    "displayname": "Provisioning Type",
    "name": "provisioningType ",
    "required": false,
    "default" :"thin",
    "type": "option"
  }
```
## Steps to register new storage template
This section describes the process to register new storage template for IBM satellite offering.
1. Raise Issue in this repository
1. Partner with IBM to initially do manual verification with:
    * Partner  Storage Driver install and config on  a Satellite Evironment
    * Identification of key configuration end user-tasks and inputs
    * Manual deployment testing across various config3.
1. Develop Satellite Storage Configuration Template to enable UI/CLI  Integration with Satellite
1. Create the fork  of this repository and commit the storage template in your fork.
1. Test the template with below command

      ` ibmcloud sat storage config create --config-name NAME --config-version VERSION --storage-template-name NAME --storage-template-version VERSION --source-branch BRANCH --source-org ORGANIZATION [-q] [--user-config-parameters PARAMETERS] [--user-secret-parameters PARAMETERS]`

      ```
      --config-name value                 Name of the storage configuration.
      --config-version value              Version of the storage configuration.
      --storage-template-name value     Specify the  storage  template name for satellite. It should match with the template name you commited.
      --storage-template-version value  Specify the  storage template version for satellite. It should match with the template version you commited.
      --source-branch The name of the branch in which you commited  your storage template 
      --source-org The name of the fork/organization in which you commited  your storage template 
      --user-config-parameters value      Specify json file path for user config parameters. To see user configuration parameters, run 'ibmcloud sat storage registered_storage get'.
      --user-secret-parameters value      Specify json file path for user secret parameters. To see user secret parameters, run 'ibmcloud sat storage registered_storage get'.
      -q                                  Do not show the message of the day or update reminders.
      ```
1. Subscribe the config created in above steps to group of satellite cluster. Check if the storage is deployed properly on the clusters. If there are any errors correct the template and  repeat the from step 3
1. Add the test result in the issue created in step 1
1. Create CI/CD test pipeline for the storage template
1. Raise a PR
1. IBM satallite team will review PR. If any change is requested, repeat  from step 3
1. IBM satallite team merges the PR
1. IBM satallite team runs the below comman to publish the storage template for other users

    `ic sat storage config template publish --name TEMPLATE_NAME --version TEMPLATE_VERSION`
    This command automatically checks of the given template is really been tested on the satellite clusters by looking up the subscrription for this template. If subscriptions are found the storage template will be published successfully. Else show the instructions/errors to satisfy the requirements for publish
1. Check if storage template is published for other users using below command 

    `ic sat storage  config template ls` 
    The newly published template will be show in the result  
1. Check if you can create storege configuration from the published template using below command. It same command as mentioned in step 5. But it does not have `source-branch` and `source-org` paramaters. It uses the published template     
       ` ibmcloud sat storage config create --config-name NAME --config-version VERSION --storage-template-name NAME --storage-template-version VERSION  [-q] [--user-config-parameters PARAMETERS] [--user-secret-parameters PARAMETERS]`
The following flow chart shows above process
![Storage Template Registration Flow](./satellite-storage-registration-flow.png) 

