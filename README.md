# ibm-satellite-storage
This repository is used to register the storage for IBM satellite. The vendor can test and create the storage template in the specified format. Once the approval cycle is complete, the storage will be available for IBM satellite offering.

# Satellite Storage Partner Certification Guideline
## Satellite overview
With IBM Cloud Satellite, you can bring your own compute infrastructure that is in your on-premises data center, at other cloud providers, or in edge networks as a Satellite Location to IBM Cloud. Then, you use the capabilities of Satellite to run IBM Cloud services on this infrastructure, and consistently deploy, manage, and control your app workloads.

Refer to:-
- [IBM Cloud Satellite](https://www.ibm.com/cloud/satellite)
- [Getting started](https://cloud.ibm.com/docs/satellite?topic=satellite-getting-started)

## Process Flowchart
![Storage Template Registration Flow](./satellite-storage-registration-flow.png) 

## How to start?
1. Create an IBMid at https://cloud.ibm.com/registration
1. Email us at *contsto2@in.ibm.com* with subject *`IBM Cloud Satellite Storage <offering name> Integration`*, example `IBM Cloud Satellite Storage LocalVolume Integration`.
   - In the email provide the high level description of the storage solution with link to user documentation or public git repository.
1. `IBM Cloud Satellite Storage` team will help you in setting up the devlopement enviorment
1. Create a single deployment yaml, in List format, to deploy Kubernetes resources for the storage operator / driver in the development enviornment.
   Following is *`sample deployment.yaml`* to deploy local-volume operator
   ```
   ---
   apiVersion: v1
   kind: List
   metadata:
       name: local-storage
       namespace: kube-system
       annotations:
           version: local-storage-45
   items:
     - apiVersion: v1 [1]
       kind: Namespace
       metadata:
           name: local-storage
     - apiVersion: operators.coreos.com/v1alpha2 [2]
       kind: OperatorGroup
       metadata:
           name: local-operator-group
           namespace: local-storage
       spec:
           targetNamespaces:
             - local-storage
     - apiVersion: operators.coreos.com/v1alpha1 [3]
       kind: Subscription
       metadata:
           name: local-storage-operator
           namespace: local-storage
       spec:
           channel: "4.5"
           installPlanApproval: Automatic
           name: local-storage-operator
           source: redhat-operators
           sourceNamespace: openshift-marketplace
     - apiVersion: local.storage.openshift.io/v1 [4]
       kind: LocalVolume
       metadata:
          name: local-disk
          namespace: local-storage
       spec:
           nodeSelector:
               nodeSelectorTerms:
                  - matchExpressions:
                        - key: storage
                          operator: In
                          values:
                              - "localvol"
           storageClassDevices:
               - storageClassName: "localblock-sc"
                 volumeMode: Block
                 devicePaths:
                    - /dev/xvde
   ```
   Above yaml defines four resource in List format
   - [1] `Namespace`
   - [2] `OperatorGroup`
   - [3] `Subscription`
   - [4] `LocalVolume`
1. Once you get your development environment, test the deployment yaml
   1. From `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Create Configuration`
   1. From `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Select the Configuration` -> `Add version and upload the deployment yaml`
   1. From IBM `Cloud Console` -> `Satellite` -> `Configurations` -> `Select the Configuration` -> `Create Subscription`
   1. Wait for few mins and check the cluster for resource deployment. 
   1. Create a PVC and run the sniff test using the IBM Provides test bucket to make sure deployment is working as expected.
1. Create an issue at https://github.com/IBM/ibm-satellite-storage/issues
1. Post the original deployment yaml, pvc yaml and test pod yaml (remove secrets from the yaml before posting) in the issue
   - Example: https://github.com/IBM/ibm-satellite-storage/issues/17
1. Post the output for
   ```
   $ os get sc
   $ os get -n <namespace> pvc
   $ os get -n <namespace> pods
   ```

## IBM Hybrid Cloud Partner Onboarding
1. Complete the onboarding process at https://ibm-10.gitbook.io/certified-for-cloud-pak-onboarding/
1. Review and fill the `Certification Requirements` checklist  https://ibm-10.gitbook.io/certified-for-cloud-pak-onboarding/co-sell-1/tech-cert/architectural-questions

## Develop Storage Configuration Template
1. Fork https://github.com/IBM/ibm-satellite-storage
1. In the deployment.yaml, created and tested above, identify the fields need to be set dynamically, like credentials, backend storage IP, device path and so on
1. Convert the `deployment.yaml` to Storage Configuration Template. 
   - Refer to [deployment.yaml](https://github.com/nkkashyap/ibm-satellite-storage/blob/local-vol/config-templates/redhat/local-volume/4.5/deployment.yaml) and [custom-parameters.json](https://github.com/nkkashyap/ibm-satellite-storage/blob/local-vol/config-templates/redhat/local-volume/4.5/custom-parameters.json)
   #### *Understanding template format*
   Let us take an example of local-volume. In case of local-volume the device path of the local storage may vary from one cluster to another. 
   To set the  device path dynamically, parameterize the device path in the template file (deployment.yaml) as follows 
   ```
     storageClassDevices:
        - storageClassName: "localblock-sc"
          volumeMode: Block
          devicePaths:
             - "{{ devicepath }}" [1]
   ```
   and define the parameter `devicepath` in `custom-parameters.json` as follows
   ```
    {
      "description": "Local storage device path", [2]
      "displayname": "Device Path", [3]
      "name": "devicepath", [4]
      "default": "/dev/sdc", [5]
      "required": true, [6]
      "type": "text" [7]
    }
   ```
   - [1] Parameter name `devicepath`, should be between `"{{ ... }}"`
   - [2] Description about the parameter
   - [3] Display name for GUI
   - [4] Name of the parameter, should be string
   - [5] Optional, default value for the parameter
   - [6] Parameter value should be set `true` | `false`
   - [7] Valid types: `text` | `secret` | `boolean` | `option` | `dropdown`

1. Maintain the following directory structure
   ```
   config-templates
   ---provider-name
      ---offering-name
         ---template-version
             ---deployment.yaml
             ---custom-parameters.json
             ---storage-class-template.yaml
             ---storage-class-parameters.json
             ---metadata.json
             ---README.me
   ```
   ```
   Example: config-templates/<provider-name>/<offering-name>/<template-version>/deployment.yaml
            config-templates/redhat/local-volume/4.5/deployment.yaml
   ```
   1. `provider-name` : The name of provider e.g IBM, AWS, Azure, Netapp, Dell/EMC
   1. `offering-name` : The storage offerig name. A provider can have multiple storage offerings for IBM satallite
   1. `template-version` : The template version. There can be multiple tempalte version for a storage offering
   1. `deployment.yaml` : Contains the Kube resources specification templates. It may include Deployment, StatefullSet, DaemonSet, Configmap, Secrets, and Storage classes in List format. 
   1. `custom-parameters.json`: This file containts the list of parameters with default value. User / admin can override the values from Sat Storage GUI. The parameter values are injected in deployment template to generate the final configuration yaml to deploy on the target clusters.
   1. `storage-class-template.yaml`: This file contains the storage class template. It eanbles creation of multiple storage classes on a satellite cluster
   1. `storage-class-parameters.json`: This file containts the list of parameters for stoarge classes. User / admin can override the values from GUI. The parameter values are injected in storage class template to generate the storage class specifications to deploy on the target clusters.
   1. `README.md`: This file contains prerequisites, deployment steps, and additional information about your template. Copy and fill out the [README_TEMPLATE.md](/.github/README_TEMPLATE.md).
   1. `metadata.json`: This file contains the metadata for GUI display.
1. Edit [template_list.json](https://github.com/nkkashyap/ibm-satellite-storage/blob/local-vol/config-templates/template_list.json) to add the new template 
   ```
    {
      "name": "local-volume",  [1] 
      "displayname": "Persistent storage using local volumes",
      "Description": "Persistent storage using local volumes for satallite cluster",
      "default-version": "4.5", [2]
      "enabled": true, [3]
      "provider": "redhat" [4]
    }
   ```
   - [1] offering-name 
   - [2] template-version
   - [3] Set it to true
   - [4] provider-name
   
1. Test the storage configuration template using CLI
   ```
   $ ibmcloud sat storage config create -h
   $ ibmcloud sat storage assignment create -h
   ```
   Example: Refer to https://github.com/IBM/ibm-satellite-storage/issues/17#issuecomment-760178811
   - Create configuration from a template in the forked repo https://github.com/nkkashyap/ibm-satellite-storage/tree/local-vol/config-templates/redhat/local-volume/4.5
   ```
   $ ibmcloud sat storage config create --name localvol-conf --template-name local-volume --template-version 4.5 -p "label-key=storage" -p "label-value=localvol" -p "devicepath=/dev/xvde" --source-org nkkashyap --source-branch local-vol
   ```
   - Apply the configuration to a cluster
   ```
   $ ibmcloud sat storage assignment create --name local-vol --cluster-group devtest --configuration localvol-conf
   ```
1. Develop test case for verification testing.
   1. Steps to setup the environment or deploy any prerequisites 
   1. Test to verify the functionality of the Storage Driver / Plugin
1. Provide runbooks for customer support with contact details
   - How customer can contact you for support?
   - Steps to collect data for debugging
   - Any known issue and any step to fix the issue.
1. Raise a PR for https://github.com/IBM/ibm-satellite-storage
   - Example: https://github.com/IBM/ibm-satellite-storage/pull/18
1. Integration with End to End Test Pipeline 
1. Final approval and publishing the template by `IBM Cloud Satellite Storage`

### Samples
#### [`deployment.yaml`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/deployment.yaml)
#### [`customer-paramerters.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/custom-parameters.json)
#### [`metadata.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/metadata.json)
#### [`storage-class-template.yaml`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/storage-class-template.yaml)
#### [`storage-class-parameters.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/storage-class-parameters.json)

