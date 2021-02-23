# IBM Spectrum Scale Container Storage Interface Driver

  

  

IBM Spectrum Scale Container Storage Interface driver allows IBM Spectrum Scale to be used as a persistent storage for stateful application running in Kubernetes clusters. Through the IBM Spectrum Scale Container Storage Interface driver, Kubernetes persistent volumes (PVs) can be provisioned from IBM Spectrum Scale. Containers can essentially be used with stateful microservices.

  

The following features are available with IBM Spectrum Scale Container Storage Interface driver:

- Static provisioning: Ability to use existing directories and filesets as persistent volumes.
- Lightweight dynamic provisioning: Ability to create directory-based volumes dynamically.
- Fileset-based dynamic provisioning: Ability to create fileset-based volumes dynamically.
- Multiple file systems support: Volumes can be created across multiple file systems.
- Remote mount support: Volumes can be created on a remotely mounted file system.
- Operator support for easier deployment, upgrade, and cleanup.
- Supported volume access modes: RWX (ReadWriteMany) and RWO (ReadWriteOnce)

## Prerequisites

  

Complete the following tasks before you start installing the IBM Spectrum Scale Container Storage Interface driver:

  - **Setup IBM Spectrum Scale Nodes for IBM Cloud Satellite**
	  -  **Setup IBM Spectrum Scale ( as root )**
		  - Follow the instructions in https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1ins_linsoft.htmto make sure all required packages are installed
			  - For example:
		  ```yum install -y kernel-devel cpp gcc gcc-c++ binutils python3```
			- Follow instructions 1-3 ( **Do NOT create the cluster** ) https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1ins_manuallyinstallingonlinux_packages.htm

	- **Setup sudo user -as root**
		- adduser, passwd
		- Follow initial instructions in https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1adm_sudowrapper.htm
		up to but not including creating a Spectrum Scale cluster
		
	- **Switch to sudo user**
		- Using the sudowrapper.htm instructions create a Spectrum Scale cluster on the worker nodes
		- verify the cluster is using sudo wrappers
		- verify Spectrum Scale starts up on all worker nodes
			```sudo /usr/lpp/mmfs/bin/mmstartup -a```
			```sudo /usr/lpp/mmfs/bin/mmgetstate -a```
		- make Spectrum Scale autostart
		```sudo /usr/lpp/mmfs/bin/mmchconfig autoload=yes```

- **Attach IBM Spectrum Scale Nodes to IBM Cloud Satellite**
	- Make sure your system is configured for the desired default route if you have more than one clustering network
	- Make sure that default route has a path to the public network, possibly via NAT or VPN

- **Start IBM Spectrum Scale Nodes**
	-	Check Spectrum Scale to see if it is running -it is likely that the IBM Cloud Openshift clustering process removed the portability layer
		-	Rebuild the portability layer if needed on each affected worker. (You may need to replace some files and subdirectories)
		```sudo yum install -y kernel-devel cpp gcc gcc-c++ binutils python3
		
		sudo mkdir /usr/include/asm
		sudo cp /usr/src/kernels/3.10.0-1160.11.1.el7.x86_64/arch/x86/include/uapi/asm/*.h
		/usr/include/asm
		
		sudo mkdir /usr/include/asm-generic
		sudo cp /usr/src/kernels/3.10.0-1160.11.1.el7.x86_64/include/uapi/asm-generic/*.h
		/usr/include/asm-generic
		
		sudo mkdir /usr/include/linux
		sudo cp /usr/src/kernels/3.10.0-1160.15.2.el7.x86_64/include/uapi/linux/*.h /usr/include/linux```

	- Start Spectrum Scale and verify it runs on all nodes
		- Follow the instructions in https://www.ibm.com/support/knowledgecenter/STXKQY_5.1.0/com.ibm.spectrum.scale.v5r10.doc/bl1adv_admrmsec.htm mount your desired filesystem remotely
			- mount the remote filesystem on all nodes and verify the mount
			- Record the information from mmcluster on both the local and remote IBM Spectrum Scale cluster(s)
- **Initialize the IBM Spectrum Scale GUI**
https://www.ibm.com/support/knowledgecenter/STXKQY_CSI_SHR/com.ibm.spectrum.scale.csi.v2r10.doc/bl1csi_instal_prereq.html
  
- **Label the Kubernetes worker nodes** where IBM Spectrum Scale client is installed and where IBM Spectrum Scale Container Storage Interface driver runs:

```

	kubectl label node node1 scale=true --overwrite=true

```
  

## IBM Spectrum Scale CSI Driver parameters & how to retrieve them

  

Retrieve all parameters required by this template.

```

	ibmcloud sat storage template get --name ess --version 1.1

```

 **ESS CSI Driver parameters**
| Parameter | Required? | Description | Default value if not provided |
| --- | --- | --- | --- |
| `scale-host-path` | Required | Mount path of the primary file system. Run: `mmlsfs`| N/A |
| `cluster-id` | Required | Cluster ID of the Primary IBM Spectrum Scale Cluster. Run `mmlscluster` from a node within the Primary Cluster.| N/A | |
| `primary-fs` | Required | Primary file system name.  Run: `mmlsfs`| N/A |
| `gui-host` | Required | FQDN or IP address of the GUI node of scale cluster that is specified against the id parameter.  Run `mmlscluster` from a node within the Primary cluster.  | N/A |
| `secret-name` | Required | Name of secret for username password to connect to Primary Cluster GUI server.  This parameter is user-specified.| N/A |
| `gui-api-user` | Required | Username to connect to Primary Cluster GUI.  Run `lsuser` from GUI node, looking for members of the CsiAdmin group.   | N/A |
| `gui-api-password` | Required | Password to connect to Primary Cluster GUI.  This parameter is user-specified.| N/A |
| `k8-n1-ip` | Required | IP address of Kubernetes Node#1 running IBM Spectrum Scale.  Run: `kubectl get nodes`| N/A |
| `sc-n1-host` | Required | Hostname of IBM Spectrum Scale Node#1.  Run: `mmlscluster`| N/A |
| `k8-n2-ip` | Required | IP address of Kubernetes Node#2 running IBM Spectrum Scale.  Run: `kubectl get nodes`| N/A |
| `sc-n2-host` | Required | Hostname of IBM Spectrum Scale Node#2.  Run: `mmlscluster`| N/A |
| `k8-n3-ip` | Required | IP address of Kubernetes Node#3 running IBM Spectrum Scale.  Run: `kubectl get nodes`| N/A |
| `sc-n3-host` | Required | Hostname of IBM Spectrum Scale Node#3.  Run: `mmlscluster`| N/A |
| `storage-class-name` | Required | Name of IBM Spectrum Scale Storage Class.  | N/A |
| `vol-backend-fs` | Required | File system on which directory-based volume should be created.  | N/A |
| `vol-dir-base-path` | Required | Base path under which all volumes with this storage class will be created.  (Must Exist).  | N/A |

## Default storage classes

| Storage class name | Type | Reclaim Policy |
| --- | --- | --- |
| `ibm-spectrum-scale-csi-lt` | light weight volumes | Delete  | 

## Creating the IBM Scale CSI Driver storage configuration

**Example `sat storage config create` command**

```sh
ibmcloud sat storage config create --name <config-name> --template-name ess --template-version 1.1 -p "scale-host-path=<scale-host-path>" -p "cluster-id=<cluseter-id>" -p "primary-fs=<primary-fs>" -p "gui-host=<gui-host>" -p "secret-name=<secret-name>" -p "gui-api-user=<gui-api-user>" -p "gui-api-password=<gui-api-password>" -p "k8-n1-ip=<k8-n1-ip>" -p "sc-n1-host=<sc-n1-host>" -p "k8-n2-ip=<k8-n2-ip>" -p "sc-n2-host=<sc-n2-host>" -p "k8-n3-ip=<k8-n3-ip>" -p "sc-n3-host=<sc-n3-host>" -p "storage-class-name=<storage-class-name>" -p "vol-backend-fs=<vol-backend-fs>" -p "vol-dir-base-path=<vol-dir-base>"
```

## Creating the storage assignment

**Example `sat storage assignment create` command**

```sh
ibmcloud sat storage assignment create --name ess --group <cluster-group-name> --config <config-name>
```

## Verifying your IBM Spectrum Scale CSI Driver storage configuration is assigned to your clusters

To verify that your configuration is assigned to your cluster. Verify that the driver pods are running, and list the Satellite storage classes that are installed.


**Example output**


## Troubleshooting

**Change Mount Point**
```
oc edit ds ibm-spectrum-scale-csi -n ibm-spectrum-scale-csi-driver
```

Search for the following string
```
/var\/lib\/kubelet$
```
Replace both instances with
```
/var/data/kubelet
```
The fix should look like this example:
```
volumeMounts:
....
-mountPath: /var/data/kubelet
	name: pods-mount-dir
....
```

**Node Mapping IBM Spectrum Scale Hosts to Kubernetes Node Names**
In some environments, Kubernetes node names might be different from the IBM Spectrum Scale node names. This results in failure during mounting of pods. Kubernetes node to IBM Spectrum Scale node mapping must be configured to address this condition during the Operator configuration.
https://www.ibm.com/support/knowledgecenter/STXKQY_CSI_SHR/com.ibm.spectrum.scale.csi.v2r10.doc/bl1csi_config_kubernet_SS_mapping_procedure_final.html


## Cleanup
Follow the instructions below to remove the IBM Spectrum Scale CSI Driver.
https://www.ibm.com/support/knowledgecenter/STXKQY_CSI_SHR/com.ibm.spectrum.scale.csi.v2r10.doc/bl1csi_operator.html

## References

- IBM Spectrum Scale CSI Driver: https://www.ibm.com/support/knowledgecenter/STXKQY_CSI_SHR/ibmspectrumscalecsi_welcome.html

