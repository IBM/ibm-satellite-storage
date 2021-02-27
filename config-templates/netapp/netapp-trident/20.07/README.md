# Netapp Trident

Trident deploys in Kubernetes clusters as pods and provides dynamic storage orchestration services for your Kubernetes workloads. It has been designed from the ground up to help you meet the persistence demands of your containerized applications by using industry standard interfaces like the Container Storage Interface (CSI).

## Prerequisites

Before installing Trident make sure that you have:
   * Access to a supported NetApp storage system.
   * Volume mount capability from all of the Kubernetes worker nodes.
   * A Linux host with `kubectl` (or `oc`, if you’re using OpenShift) installed and configured to manage the clusters that you want to use.

## Netapp Trident Driver parameters & how to retrieve them

The `netapp-trident` template is used to install Trident in OpenShift clusters. As such, this template does not expose any Trident storage driver parameters. In a nutshell, the workflow used to deploy Trident and have it communicate with ONTAP clusters will look like this:

1. Install the `netapp-trident`. When you create a Satellite storage configuration that uses the `netapp-trident` template and assign it to your Satellite groups, the Trident storage drivers are deployed on your clusters. After you verify that Trident is up and running, you can deploy the `netapp-ontap-san` or `netapp-ontap-nas` templates which allow your clusters to interact with your NetApp backends.
2. Deploy the `netapp-ontap-san` and/or `netapp-ontap-nas` storage templates. As part of this step, a [Trident backend](https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/index.html) is created for the respective protocol and configured based on the template parameters provided. This step also defines multiple storageClasses to be consumed by the OpenShift user.
3. Trident is now installed, configured, and ready to use!

## Default storage classes

---
**Note**
   
Deploying Trident does not create StorageClasses for you. This is handled by deploying the `netapp-ontap-san` or `netapp-ontap-nas` storage templates. This template purely installs Trident for you. Proceed with the installation of the `netapp-ontap-nas` and/or `netapp-ontap-san` storage templates to have storage classes created automatically.

---
     
## Creating the Netapp Trident storage configuration


**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name "trident-conf" --template-name "netapp-trident" --template-version "20.07"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name "trident-operator --group <group name> --config "trident-conf"
```

## Verifying your Netapp Trident configuration is assigned to your clusters.

```sh
oc get all -n trident
```

**Example output**

```sh
NAME                                    READY   STATUS    RESTARTS   AGE
pod/trident-csi-68d979fb85-dsrmn        6/6     Running   0          1d
pod/trident-csi-8jfhf                   2/2     Running   0          1d
pod/trident-csi-jtnjz                   2/2     Running   0          1d
pod/trident-csi-lcxvh                   2/2     Running   0          1d
pod/trident-operator-7569744d4f-hwgnr   1/1     Running   0          1d

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
service/trident-csi   ClusterIP   10.108.174.125   <none>        34571/TCP,9220/TCP   1d

NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                     AGE
daemonset.apps/trident-csi   3         3         3       3            3           kubernetes.io/arch=amd64,kubernetes.io/os=linux   1d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/trident-csi        1/1     1            1           1d
deployment.apps/trident-operator   1/1     1            1           1d

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/trident-csi-68d979fb85        1         1         1       1d
replicaset.apps/trident-operator-7569744d4f   1         1         1       1d
```

After succesfully installing the template, you should have multiple Trident pods running in the `trident` namespace.

## Troubleshooting

### Trident pod stuck in `ContainerCreating` status

If the trident pod fails deploy and is stuck in `ContainerCreating` status, you can describe the Trident resources to gain additional information.

1. Describe the deployment by running the `oc -n trident describe deployment trident` command and review the command output.
   ```sh
   oc -n trident describe deployment trident
   ```
2. Describe the Trident pod by running `oc -n trident describe pod trident-********-****` command and review the command output.
   ```sh
   oc -n trident describe pod trident-********-****
   ```
3. You can also use `tridentctl`, Trident's binary tool, to obtain logs and pinpoint the source of the problem. You can download `tridentctl` from [Trident's GitHub repo](https://github.com/NetApp/trident/releases) for the appropriate version of Trident. Run the following command and look for problems in the logs for the `trident-main` and CSI containers.
   ```sh
   tridentctl logs -l all -n trident
   ```
4. You can also run `oc logs` to retrieve the logs for the `trident-********-****` pod.
   ```sh
   oc logs trident-********-****
   ```
 

## Reference

- [Trident documentation](https://netapp-trident.readthedocs.io/en/stable-v20.07/introduction.html)
- [ONTAP NAS backend configuration](https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-nas/index.html)
- [ONTAP SAN backend configuration](https://netapp-trident.readthedocs.io/en/stable-v20.07/kubernetes/operations/tasks/backends/ontap/ontap-san/index.html)
- [Support documentation](https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html)
