# IBM Satellite storage
This repository is used to develop storage templates to install your vendor storage solution in clusters that run in  IBM Cloud Satellite. As a vendor, you can create and test your storage templates by following this readme. After your template is approved, it can be deployed to clusters that run in IBM Cloud Satellite.

# Satellite Storage Partner Certification Guideline
## Satellite overview
With IBM Cloud Satellite, you can bring your own compute infrastructure that is in your on-premises data center, at other cloud providers, or in edge networks as a Satellite Location to IBM Cloud. Then, you use the capabilities of Satellite to run IBM Cloud services on this infrastructure, and consistently deploy, manage, and control your app workloads. For more information, see the IBM documentation.
- [IBM Cloud Satellite](https://www.ibm.com/cloud/satellite)
- [Getting started](https://cloud.ibm.com/docs/satellite?topic=satellite-getting-started)

## Process Flowchart
![Storage Template Registration Flow](./satellite-storage-registration-flow.png) 

## Registering with IBM Cloud
1. [Create an IBMid](https://cloud.ibm.com/registration).
1. Email `contsto2@in.ibm.com`.
   1. Include the subject line: `IBM Cloud Satellite Storage <storage-solution-name> Integration`*. For example: `IBM Cloud Satellite Storage LocalVolume Integration`. 
   1. Provide a high level description of the storage solution with a link to the user documentation or public repository.
1. [Complete the onboarding process](https://ibm-10.gitbook.io/certified-for-cloud-pak-onboarding)
1. [Complete the Certification Requirements checklist.](https://ibm-10.gitbook.io/certified-for-cloud-pak-onboarding/co-sell-1/tech-cert/architectural-questions)



## Creating and testing your deployment

1. Create a `deployment.yaml` file to deploy Kubernetes resources for the storage operator or driver in the development enviornment. Review the following example `deployment.yaml` to deploy the `local-volume` operator.
   ```yaml
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
1. [Create an issue in this repo.](https://github.com/IBM/ibm-satellite-storage/issues). Review the formatting of the [example issue](https://github.com/IBM/ibm-satellite-storage/issues/17). Include the following information in the issue. Ensure that you remove any secrets from your files.
   1. Your `deployment.yaml` file.
   1. An example`pvc.yaml` file.
   1. An example `app.yaml`.
   1. The output of the `oc get sc` command that displays the storage classes your deployment creates.
   1. The output of the `oc get -n <namespace> pvc` command that displays the PVCs your deployment creates.
   1. The output of the `oc get -n <namespace> pods` command that displays the pods your deployment creates.
   ```

## Developing your Satellite storage configuration template
1. Fork this repository. https://github.com/IBM/ibm-satellite-storage
1. In your `deployment.yaml`, identify the fields that need to be set dynamically. For example: credentials, backend storage IP, or device paths.
1. Convert the `deployment.yaml` to a Satellite storage configuration template. Refer to the example [deployment.yaml](https://github.com/nkkashyap/ibm-satellite-storage/blob/local-vol/config-templates/redhat/local-volume/4.5/deployment.yaml) and [custom-parameters.json](https://github.com/nkkashyap/ibm-satellite-storage/blob/local-vol/config-templates/redhat/local-volume/4.5/custom-parameters.json) files.


### Understanding template format
Let us take an example of local-volume. In case of local-volume the device path of the local storage may vary from one cluster to another. 
To set the device path dynamically, parameterize the device path in the template `deployment.yaml` file in the following format.
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

## Testing and support
1. Develop test case for verification testing. You test case should include the following:
   1. Steps to set up the environment or deploy any prerequisites.
   1. Test to verify the functionality of the Storage driver.
1. Provide a runbook for customer support. Include the following information in your support runbook.
   - Support contact details.
   - Link to support site or support ticket process. 
   - Steps to collect data for debugging.
   - Known issues and steps to resolve the issue.

## Requesting review
After you have completed testing, you can open a PR to request a review from the Satellite storage team.

1. Create a PR from your fork to the `develop` branch of this repo.
1. The Satellite storage team will review your PR and request changes or approve.

## Example storage configuration template resources
- [`deployment.yaml`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/deployment.yaml)
- [`customer-paramerters.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/custom-parameters.json)
- [`metadata.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/metadata.json)
- [`storage-class-template.yaml`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/storage-class-template.yaml)
- [`storage-class-parameters.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/storage-class-parameters.json)

