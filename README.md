# IBM Cloud Satellite storage
This repository is used to develop storage templates to install your vendor storage solution in clusters that run in IBM Cloud Satellite. As a vendor, you can create and test your storage templates by following this README. After your template is approved, it can be deployed to clusters that run in IBM Cloud Satellite.

## Overview
With IBM Cloud Satellite, you can bring your own compute infrastructure that is in your on-premises data center, at other cloud providers, or in edge networks as a Satellite Location to IBM Cloud. Then, you use the capabilities of Satellite to run IBM Cloud services on this infrastructure, and consistently deploy, manage, and control your app workloads. For more information, see the IBM documentation.
- [IBM Cloud Satellite](https://www.ibm.com/cloud/satellite)
- [Getting started](https://cloud.ibm.com/docs/satellite?topic=satellite-getting-started)

## Satellite storage partner certification process
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
1. Once you get your development environment, test your deployment.
   1. From `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Create Configuration`
   1. From `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Select the Configuration` -> `Add version and upload the deployment yaml`
   1. From the `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Select the Configuration` -> `Create Subscription`
   1. Verify that the resources are deployment to the cluster. 
   1. Create a PVC and run the sniff test using the IBM Provides test bucket to make sure deployment is working as expected.
 
1. After you have verified your template in the development environment, [create an issue in this repo.](https://github.com/IBM/ibm-satellite-storage/issues). Review the formatting of the [example issue](https://github.com/IBM/ibm-satellite-storage/issues/17). Include the following information in your issue and ensure that you remove any secrets from your files.
   1. Your `deployment.yaml` file.
   1. An example`pvc.yaml` file.
   1. An example `app.yaml`.
   1. The output of the `oc get sc` command that displays the storage classes your deployment creates.
   1. The output of the `oc get -n <namespace> pvc` command that displays the PVCs your deployment creates.
   1. The output of the `oc get -n <namespace> pods` command that displays the pods your deployment creates.

## Developing your Satellite storage configuration template
1. Fork this repository.
1. In your `deployment.yaml`, 
1. Convert the `deployment.yaml` to a Satellite storage template.
1. In your fork, make sure that you create your directory structure in the following format. For more information about the template files, review the reference table.
   ```md
   config-templates
   ---<storage-provider-name> 
      ---<storage-offering-name>
         ---<template-version>
               ---deployment.yaml
               ---custom-parameters.json
               ---storage-class-template.yaml
               ---storage-class-parameters.json
               ---metadata.json
               ---README.md
   ```

### File reference

| `storage-provider-name` | The name of storage provider. Example: `ibm`, `aws`, `azure`, `netapp`, `dell`. |
| `storage-offering-name` | The storage offerig name. A provider can have multiple storage offerings for IBM satallite. |
| `template-version` | The template version. There can be multiple tempalte version for a storage offering |
| `deployment.yaml` | A custom Kubernetes `List` that includes the resources like Deployment, StatefulSet, DaemonSet, Configmap, secrets, and storage classes. Example [`deployment.yaml`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/deployment.yaml). |
| `custom-parameters.json` | This file contains the list of parameters that your deployment accepts. Include any required and optional paramters and their default values. Example [`customer-paramerters.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/custom-parameters.json). |
| `storage-class-template.yaml` | The storage class template. This template can be used to create storage classes on a Satellite cluster. Example [`storage-class-template.yaml`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/storage-class-template.yaml). |
| `storage-class-parameters.json` | The list of storage class parameters. User / admin can override the values from GUI. The parameter values are injected in storage class template to generate the storage class specifications to deploy on the target clusters. [`storage-class-parameters.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/storage-class-parameters.json) |
| `README.md` | Contains prerequisites, deployment steps, and additional information about your template. Copy and fill out the [README_TEMPLATE.md](/.github/README_TEMPLATE.md). |
| `metadata.json` | This file contains the metadata for GUI display. Example [`metadata.json`](https://github.com/IBM/ibm-satellite-storage/blob/master/config-templates/netapp/netapp-trident/20.07/metadata.json). |


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



## Testing and support
1. Create a configuration from a template in your fork. When you run the `sat storage config create` command, specify the `source-org` and `source-branch` of your fork and any other parameters for your configuration.
   ```sh
   ibmcloud sat storage config create --name <config-name> --template-name <template-name> --template-version <template-version> --source-org <your-github-org> --source-branch <your-github-branch> [-p "<parameter>=<value>" -p "<parameter>=<value>"]
   ```

1. Assign your configuration to a cluster.
   ```sh
   ibmcloud sat storage assignment create --name <assignment-name> --group <cluster-group> --configuration <config-name>
   ```
1. Develop test cases for verification testing. You test case should include the following:
   * Steps to set up the environment or deploy any prerequisites.
   * Test to verify the functionality of the Storage driver.
1. Provide a runbook for customer support. Include the following information in your support runbook.
   * Support contact details.
   * Link to support site or support ticket process. 
   * Steps to collect data for debugging.
   * Known issues and steps to resolve the issue.
## Requesting review
After you have completed testing, you can open a PR to request a review from the Satellite storage team.

1. Create a PR from your fork to the `develop` branch of this repo.
1. The Satellite storage team will review your PR and request changes or approve.
