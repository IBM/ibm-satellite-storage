# Development of storage class template 
The storage class template is created in `storage-class-template.yaml` file as per the directory structure mentioned [here]README.md#filedirectory-reference).
This file defines the template that contain the configuration of storage classes in YAML format. It is Kubernetes `StorageClass` resource that contains the set of storage class parameters and allowed variable substitutions.  The  example of `storage-class-template.yaml` is as below 
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: "{{ name }}"
  annotations:
    storageclass.kubernetes.io/is-default-class: "{{ is-default-class }}"
provisioner: block.csi.ibm.com
parameters:
  SpaceEfficiency: "{{ space-efficiency }}"   # Optional.
  pool: "{{ pool }}" 

  csi.storage.k8s.io/provisioner-secret-name: "{{ secret-name }}"
  csi.storage.k8s.io/provisioner-secret-namespace: "{{ secret-namespace }}"
  csi.storage.k8s.io/controller-publish-secret-name: "{{ secret-name }}"
  csi.storage.k8s.io/controller-publish-secret-namespace: "{{ secret-namespace }}"
  csi.storage.k8s.io/controller-expand-secret-name: "{{ secret-name }}"
  csi.storage.k8s.io/controller-expand-secret-namespace: "{{ secret-namespace }}"

  csi.storage.k8s.io/fstype: "{{ fstype }}"   # Optional. Values ext4\xfs. The default is ext4.
  volume_name_prefix: "{{ prefix }}"      # Optional.
allowVolumeExpansion: {{ VolumeExpansion }} #false
```

We DO NOT use `MustachTemplate` engine to translate the storage class template to actual storage class configuration. Rather, we just replace  ` {{ varaibale-name }}` with its value given by the user. The multiple storage classes can be created from the same storage-class-template. The user is allowed to pass the array of storage class parameters to create multiple storage classes. The triple curly bracket notation e.g ` {{{ varaibale-name }}}`  is not allowed in storage class template . Its only allowed in `deployement.yaml`.

## Customer parameter requirements for storage class
   The custom parameters for the storage class are mentioned in the `storage-class-parameters.json` file It follows the same syntax as customer parameters for the deployment template. The existing example can be found [here](config-templates/ibm/ibm-block-storage-csi-driver/1.4.0/storage-class-parameters.json). The `name` and `is-default-class` are the mandatory custom parameters that need to be defined for storage class template. This allows user to create multiple storage classes without conflicting the `storage class name` and the `default storage class`
   
## Configuration of Single Valued variable
   This is a simple variable with single value of type `text` , `boolean` or `number`. In the following example , `pool` , `secret-name `  and `secret-namespace` are the single valued variables. 

__Template YAML__
   ``` 
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: "{{ name }}"
      annotations:
        storageclass.kubernetes.io/is-default-class: "{{ is-default-class }}"
    provisioner: block.csi.ibm.com
    parameters:
      SpaceEfficiency: "{{ space-efficiency }}"   # Optional.
      pool: "{{ pool }}" 
      csi.storage.k8s.io/provisioner-secret-name: "{{ secret-name }}"
      csi.storage.k8s.io/provisioner-secret-namespace: "{{ secret-namespace }}"
         
  ```

__User Parameteter__
   name=my-storage-class
   is-default-class=false
   SpaceEfficiency=test
   pool=testpool
   secret-name=mysec
   secret-namespace=mynamespace


__Translated YAML__

```      
   -apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: "my-storage-class"
      annotations:
        storageclass.kubernetes.io/is-default-class: "false"
    provisioner: block.csi.ibm.com
    parameters:
      SpaceEfficiency: "test"
      pool: "test-pool"
      csi.storage.k8s.io/provisioner-secret-name: "mysec"
      csi.storage.k8s.io/provisioner-secret-namespace: "mynamespace"

``` 


## Configuration of empty variable
   The optional custom parameters might not be given by the user and they will remain empty if no default value is available. In the following example `space-efficiency` is optional parameter and user do not provide any value . In this case, the empty valued parameter is removed from the actual storage class.

__Template YAML__
   ``` 
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: "{{ name }}"
      annotations:
        storageclass.kubernetes.io/is-default-class: "{{ is-default-class }}"
    provisioner: block.csi.ibm.com
    parameters:
      SpaceEfficiency: "{{ space-efficiency }}"   # Optional.
      pool: "{{ pool }}" 
      csi.storage.k8s.io/provisioner-secret-name: "{{ secret-name }}"
      csi.storage.k8s.io/provisioner-secret-namespace: "{{ secret-namespace }}"
         
  ```

__User Parameteter__
   name=my-storage-class
   is-default-class=false
   pool=testpool
   secret-name=mysec
   secret-namespace=mynamespace


__Translated YAML__

```      
   -apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: "my-storage-class"
      annotations:
        storageclass.kubernetes.io/is-default-class: "{{ is-default-class }}"
    provisioner: block.csi.ibm.com
    parameters:
      pool: "test-pool"
      csi.storage.k8s.io/provisioner-secret-name: "mysec"
      csi.storage.k8s.io/provisioner-secret-namespace: "mynamespace"

``` 

## Vendor defined storage classes
   The vendor defined storage classes are configured in the `storage-class.yaml` as per the directory structure mentioned [here]README.md#filedirectory-reference). The existing example of vendor defined storage classes can be found [here](config-templates/netapp/netapp-ontap-nas/20.07/storage-class.yaml). These classes will be readily available for the user to create the storage volumes. 

### Merging User defined classes with Vendor defined storage classes 
  Vendor can have both vendor defined storage classes in `storage-class.yaml` and storage class template in `storage-class-template.yaml`. This allows user to create his own storage classes and still use the vendor defined storage classes. If user given storage class name conflicts with vendor provided storage class, it will give preference to user defined storage class and override the definition of vendor defined storage class. This will enable the user to change the `default` storage class

  