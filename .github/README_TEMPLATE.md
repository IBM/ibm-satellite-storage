# Provider name / Configuration name



## Prerequisites

List any config steps to take before creating a configuration using with your template. For example: resource requirements such as volumes, nodes, supported versions, etc.

## Required / Optional parameters & how to retrieve them

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


## Installation steps

Provide an example steps, including an example `config create` command for creating a Satellite storage configuration that uses your template.

**Example config create**

```sh
ibmcloud sat storage config create --name <example-config-name> --template-name <template-name> --template-version <template-version> -p "<parameter-name>=<parameter-value>"
```

