# IBM block storage Container Storage Interface driver

IBM block storage CSI driver is based on an open-source IBM project, included as a part of IBM Storage orchestration for containers. IBM Storage orchestration for containers enables enterprises to implement a modern container-driven hybrid multicloud environment that can reduce IT costs and enhance business agility, while continuing to derive value from existing systems.

For full release notes, compatiblity, installation, and user information, see the IBM block storage CSI driver documentation: https://www.ibm.com/support/knowledgecenter/SSRQ8T_1.4.0/

## Prerequisites

For full prerequisite instructions, see the following documentation: Installation > Compatibility and requirements.

**Important:** Be sure to complete all prerequisite and installation steps before assigning hosts to your location. Do not create a Kubernetes cluster. This is done through Satellite.

## IBM block storage CSI driver parameters & how to retrieve them



Retrieve all parameters required by this template.

```

	ibmcloud sat storage template get --name ibm-csi-block --version 1.4.0

```

 **IBM block storage CSI driver parameters**
| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `namespace` | Optional | Deployment Namespace. | default |


## Creating the IBM block storage CSI driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <config-name> --template-name ibm-csi-block --template-version 1.4.0 -p "namespace=<namespace>" "
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name <assignment-name> --group <cluster-group-name> --config <config-name>
```

## Verifying your IBM block storage CSI driver storage configuration is assigned to your clusters

To verify that your configuration is assigned to your cluster. Verify that the driver pods are running.


**Example output**


## Troubleshooting


## References

- IBM block storage CSI driver documentation: https://www.ibm.com/support/knowledgecenter/SSRQ8T
