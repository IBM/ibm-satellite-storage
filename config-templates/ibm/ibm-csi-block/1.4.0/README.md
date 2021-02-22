# IBM block storage Container Storage Interface Driver



## Prerequisites



## IBM block storage CSI Driver parameters & how to retrieve them



Retrieve all parameters required by this template.

```

	ibmcloud sat storage template get --name ibm-csi-block --version 1.4.0

```

 **IBM block storage CSI Driver parameters**
| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `namespace` | Optional | Deployment Namespace. | default |


## Creating the IBM block storage CSI Driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <config-name> --template-name ibm-csi-block --template-version 1.4.0 -p "namespace=<namespace>" "
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name <assignment-name> --group <cluster-group-name> --config <config-name>
```

## Verifying your IBM Spectrum Scale CSI Driver storage configuration is assigned to your clusters

To verify that your configuration is assigned to your cluster. Verify that the driver pods are running.


**Example output**


## Troubleshooting


## References

- [IBM knowledge center][https://www.ibm.com/support/knowledgecenter/SSRQ8T_1.4.0/csi_block_storage_kc_welcome.html]
