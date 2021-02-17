---
layout: default
description: IBM Satellite Storage NetApp Trident
title: Trident is not getting deployed in a Satellite Cluster
service: Satellite Storage
runbook-name: "Trident is not getting deployed in a Satellite Cluster"
---


## Overview

This runbook provides resolution for common issues while deploying the Trident and Ontap-NAS driver in a Satellite cluster.


---

## Investigation and Action

#### Issue: After assigning the configuration to a Cluster, Trident and the Ontap-nas PODs are not getting deployed under the target namespace.

1. Login into the Cluster using `OC CLI`
   - Login to [IBM Cloud Web Console](https://cloud.ibm.com/)
	 - Go to [Clusters page](https://cloud.ibm.com/satellite/clusters)
   - Client on the row with the cluster details, You will get your cluster page
   - Click on `Manage Cluster` and in next page click on `OpenShift web console`
   - In OpenShift console click on your user ID at top right corner and then click on `Copy Login Command`
   - In next page Click on `Display Token`
   - Under `Log in with this token` the `oc login --token=XXXX ...` will be displayed, copy the command and execute on your local system

2. Check the status of the resource under `razeedeploy` namespace
   - Check the status of the resources  
     ```
     $ oc get all -n razeedeploy
     ```
   - if the output for above command is
     ```
     No resources found in razeedeploy namespace.
     ```
     then go to Step 3 and re-attach the cluster
     otherwise go to Step 5

3. Re-Attach the cluster
   - In IBM Cloud Web Console go to [Clusters page](https://cloud.ibm.com/satellite/clusters)
	 - In the cluster row click on action menu (three vertical dots at end) 
	 - Select `Re-attach cluster`
   - Copy the command to clipboard
   - Execute the command from your local system

4. Check the status of the resources  
    ```
    $ oc get all -n razeedeploy
    ```
    if any POD go to error/unknown/any state other than 'Ready' then from IBM Cloud console select `Support` and raise support ticket.

5. Check the status of Trident and Driver resource the target namespace
   ```
   $ oc get all -n trident
   ```
   if any POD go to error/unknown/any state other than 'Ready' then contact [NetApp Support](https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html).
---

## Automation

None

---

## Escalation Policy

* Please contact [NetApp Support](https://netapp-trident.readthedocs.io/en/stable-v20.10/support/support.html) for any other issue with Ontap-NAS deployment.



---
