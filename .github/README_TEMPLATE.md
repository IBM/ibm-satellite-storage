# [Template Name]

Enter a short description of your template and storage provider.

## Prerequisites

List any config steps to take before creating a configuration with your template. For example, your prerequisite steps might include any or all of the following.

- Resource requirements such as volumes, nodes, disks.
- Supported versions such as compatible OCP versions

## [Template Name] parameters & how to retrieve them

Create a table that contains the required and optional parameters that are used when creating a Satellite storage configuration with your template. For example your configuration might require or accept any or all of the following parameters.

- Secrets or passwords
- Labels
- Mount points or device paths

**[Template Name] parameters**

Note that each parameter in your parameter table should include the following details:

- Parameter name.
- Whether or not the parameter is required.
- How to retrieve the parameter. 
- The default value of the parameter (if applicable).

| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `parameter` | Required | This is a required parameter. You can retrieve this parameter by... | N/A |
| `parameter-2` | Optional | This is an optional parameter. | `param` |
| `parameter-3` | Conditional | This parameter is conditional on another parameter. For example: `parameter-3` is required if `parameter-2` is specified. | N/A |


## Default storage classes

Provide a table of the Satellite storage classes that are installed when your configuration is assigned to a cluster. Refer to the following example table. Add or remove storage class details based on the storage type.

| Storage class name | Type | File system | IOPs | Size range | Hard disk | Reclaim policy |
| --- | --- | --- | --- | --- | --- |
| `sat-storage-class-name-delete-bronze` | Endurance | NFS | 2 | 20-12000 Gi | SSD | Delete | 


## Creating the [Template Name] storage configuration

Provide example steps, including an example `config create` command for creating a Satellite storage configuration that uses your template.

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <example-config-name> --template-name <template-name> --template-version <template-version> -p "<parameter-name>=<parameter-value>"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name <assignment-name> --cluster-group <cluster-group> --configuration <configuration-name>
```

## Verifying your [Template Name] storage configuration is assigned to your clusters

Provide steps to retrieve any driver pods, storage classes, PVs, PVCs, or any other resources that are deployed with your storage configuration.

```sh
oc get pods 
```

```sh
oc get pv
```

```sh
oc get pv
```

```sh
oc get sc | grep sat
```


**Example output**

Provide an example output screen of running driver pods and storage classes that are deployed by your configuration.

## Deploying an app that uses your storage configuration
 - Provide an example PVC yaml.
 - Provide an example pod or deployment yaml.

### Verifying that your app can write data to your storage
- Provide steps to log-in to the app pod and write a file to the storage instance.


## Removing your assignment

Provide steps to remove the assignment from clusters.

**What happens when I remove the <template-name> assignment from my clusters?**
 - Which resources get removed? (pods, pvcs, deployments, storage classes, etc.)
 - What happens to my data? Is it deleted? Do my exisiting apps still work?

**Example `assignment rm` command

```sh
ibmcloud sat storage assignment rm
```

## Removing your configuration

Provide steps and example command to remove the storage configuration.

```sh
ibmcloud sat storage configuration rm
```


## Cleaning up your deployment

 Provide clean up instructions - how to delete all resources, pods, etc. Delete any resources that are left behind after deleting the assignment.

## Troubleshooting

Provide troubleshooting steps for any known issues.


## Support and "must gather" information

**What information should I gather before opening a ticket?**
- Provide a list of the must gather information.

**How do I gather the required information?**

1. Steps to gather logs, check pod health, etc.

## Reference

- Include a link to your provider documentation.
- Limitations specific to Satellite.
- Include a link to the support process documentation for your provider.