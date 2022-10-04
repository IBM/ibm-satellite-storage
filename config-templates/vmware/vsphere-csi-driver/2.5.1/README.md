# VMware CSI Driver via VDO

vSphere CSI Driver is a way to leverage the VMware storage services on Containerized solutions. This templates brings in an easy way to deploy vSphere CSI driver through an operator VDO.  
VDO helps in configuring the CSI driver in an easy way and manage the lifecycle of the driver.

## Contributing Guide
To bring in any changes in the template you need to run the driver against the latest roks on IBM Satellite.  
Easy way to start with this is to create vm's in vCenter and then attach this to cloud. Below are VM's specs.
- vCenter which is public to internet. 
- 6 VM's with 4 CPU , 16GB Memory and 200 Gb Hard-disk.   
- These machines should be publicly accessible. 
- A Redhat account. 
- Access to NSX setup manager. 
- Rhel 7 ISO. 

To spawn new VM's you need to go to vCenter and follow the steps
VC → inventory→ Choose DataCenter → IBM Cloud Satellite→ Create a new virtual machine → 
Use vSphere 6.7 or later → RHEL 7 → 4cpu → 16 GB Memory→ 200GB Harddisk → Network(ibm07) → Datastore ISO, RHEL 7.9 iso 
PowerOn the machine and login to web console, follow the installation step, and configure the network as shown above.
Reboot the machine and follow the below steps
```
subscription-manager register
subscription-manager attach --auto
yum check-update
yum update   <=== This is important!

sudo subscription-manager refresh
sudo subscription-manager repos --enable rhel-server-rhscl-7-rpms
sudo subscription-manager repos --enable rhel-7-server-optional-rpms
sudo subscription-manager repos --enable rhel-7-server-rh-common-rpms
sudo subscription-manager repos --enable rhel-7-server-supplementary-rpms
sudo subscription-manager repos --enable rhel-7-server-extras-rpms

Then run the script you copied to your VMs
sudo nohup bash /tmp/attach.sh &
```

You should assign hosts to control plane. Each zone must have at least 1 node for the control plane.

Click on the location -> hosts -> click on three dots -> Assign.
Choose control plane and zone and hit assign. 

Repeat this for three nodes.
Note: For every node choose a different zone

The remaining three nodes would be assigned to cluster. You can create the cluster only when the location is in ready state.

If you dont assign then nodes in all three zones then the location would not come into ready state
