# AZURE DISK CSI Driver[BETA]

Azure Disk CSI driver implements the CSI specification for container orchestrators to manage the lifecycle of Azure Disk volumes.

**Features supported:**
- Topology(Availability Zone)
    - ZRS disk support(Preview)
- Volume Cloning
- Volume Expansion
- Raw Block Volume
- Shared Disk
- Volume Limits
- fsGroupPolicy

## Prerequisites
1. Retrieve the zone of your Azure worker nodes.
    ```
    oc get nodes
    ```
2. Label your Azure worker nodes with the zone. Replace `<zone>` with the zone of your Azure worker nodes. For example: `eastus-1`.
    ```
    oc label node <node_name> topology.kubernetes.io/zone-
    oc label node <node_name> topology.kubernetes.io/zone=<zone> --overwrite
    ```
3. Copy the [Azure disk configuration template](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/azure.json) and enter the details for your cluster.
4. Serialize your config file and convert it to base64.
    ```
    cat azure.json | base64 | awk '{printf $0}'; echo
    ```


## Azure Disk CSI Driver parameters & how to retrieve them

Get a list of the Azure template configuration parameters. 
```
ibmcloud sat storage template get --name azuredisk-csi-driver --version 1.4.0
```

** Azure Disk CSI Driver parameters**

| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `cloud-config` | Required | Enter the serialized value that your created from the `azure.json` [conifg file](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/azure.json). | N/A |


## Default storage classes

| Storage class name | IOPS range per disk | Size range | Hard disk | Reclaim policy | Volume Binding Mode |
| --- | --- | --- | --- | --- | --- |
| `sat-azure-block-platinum` |  1200 - 160000 | 4 GiB - 64 TiB | SSD | Delete | Immidiate |
| `sat-azure-block-platinum-metro`  | 1200 - 160000 | 4 GiB - 64 TiB | SSD | Delete | WaitForFirstConsumer |
| `sat-azure-block-gold` | 120 - 20000 | 32 GiB - 32 TiB | SSD | Delete | Immidiate |
| `sat-azure-block-gold-metro` | 120 - 20000 | 32 GiB - 32 TiB | SSD | Delete | WaitForFirstConsumer |
| `sat-azure-block-silver`  | 120 - 6000 | NA | SSD | Delete | Immidiate |
| `sat-azure-block-silver-metro` | 120 - 6000 | NA | SSD | Delete | WaitForFirstConsumer |
| `sat-azure-block-bronze`  | 500 - 2000 | 32 GiB - 32 TiB | HDD | Delete | Immidiate |
| `sat-azure-block-bronze-metro` | 500 - 2000 | 32 GiB - 32 TiB | HDD | Delete | WaitForFirstConsumer |



## Creating the AZURE DISK CSI Driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <config-name> --template-name azuredisk-csi-driver --template-version 1.4.0 --location <location> -p "cloud-config=<base64-encoded-config-file>"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

**Apply config to a group of clusters**
```sh
ibmcloud sat storage assignment create --name <assignmemt-name> --group <cluster-group> --config <config-name>
```
**Apply config to an individual cluster**
```sh
ibmcloud sat storage assignment create --name <assignmemt-name> --cluster <cluster-id> --config <config-name>
```
**Apply config to a service cluster**
```sh
ibmcloud sat storage assignment create --name <assignmemt-name> --service-cluster-id <service-cluster-id> --config <config-name>
```
## Verifying your Azure Disk CSI Driver storage configuration is assigned to your clusters

To verify that your configuration is assigned to your cluster. Verify that the driver pods are running, and list the Satellite storage classes that are installed.

List the `azuredisk` driver pods in the `kube-system` namespace and verify that the status is `Running`.

```
% kubectl get pods -n kube-system | grep azure
csi-azuredisk-controller-849d854b96-6jbjg   5/5     Running   0          167m
csi-azuredisk-controller-849d854b96-lkplx   5/5     Running   0          167m
csi-azuredisk-node-7qwlj                    3/3     Running   6          167m
csi-azuredisk-node-8xm4c                    3/3     Running   6          167m
csi-azuredisk-node-snsdb                    3/3     Running   6          167m
```

List the `azuredisk` storage classes.

```
% kubectl get sc | grep azure
sat-azure-block-bronze           disk.csi.azure.com   Delete          Immediate              true                   167m
sat-azure-block-bronze-metro     disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   167m
sat-azure-block-gold             disk.csi.azure.com   Delete          Immediate              true                   167m
sat-azure-block-gold-metro       disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   167m
sat-azure-block-platinum         disk.csi.azure.com   Delete          Immediate              true                   167m
sat-azure-block-platinum-metro   disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   167m
sat-azure-block-silver           disk.csi.azure.com   Delete          Immediate              true                   167m
sat-azure-block-silver-metro     disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   167m
```

**Example output**

![Example Output](./images/output.png)

## Troubleshooting
- In case of `node register failure`, please make sure that nodes are labelled with proper zone.
- In case of `authentication failure`, please make sure that **Service Principal** is created properly.

## References
[Azure Disk CSI Driver](https://github.com/kubernetes-sigs/azuredisk-csi-driver)
[Limitations](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/docs/limitations.md)
