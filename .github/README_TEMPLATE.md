# Storage provider name or the template name

Enter the name of the storage provider or the name of your storage template

## Prerequisites

List any config steps to take before creating a configuration using with your template. For example: resource requirements such as volumes, nodes, supported versions, etc.

## Parameters & how to retrieve them

Create a table that contains the required and optional parameters that are used when creating a Satellite storage configuration with your template.

**Example parameter table**

Note that each parameter in your parameter table should include the following details:

- Parameter name
- Whether or not the parameter is required
- How to retrieve the parameter. 
- The default value of the parameter (if applicable).

| Parameter | Description | Default value if not provided |
| --- | --- | --- |
| `parameter` | Required. This is a required parameter. You can retrieve this parameter by... | N/A |
| `parameter-2` | Optional. This is an optional parameter. | `param` |


## Creating the storage configuration

Provide an example steps, including an example `config create` command for creating a Satellite storage configuration that uses your template.

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <example-config-name> --template-name <template-name> --template-version <template-version> -p "<parameter-name>=<parameter-value>"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name <assignment-name> --cluster-group <cluster-group> --configuration <configuration-name>
```

## Verification steps

Provide steps to retrieve driver pods, storage classes, or any other resources that are deployed with your template.

```sh
oc get
```

**Example output**

Provide an example output screen of running driver pods and storage classes that are deployed by your configuration.