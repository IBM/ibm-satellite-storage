# Development of deployment template 
The deployment template is created in `deployment.yaml` file as per the directory structure mentioned [here](README.md#File/Directory)
This file defines the template that contain the configuration of storage solution in YAML format. It is Kubernetes `List` that includes the resources like Deployment, StatefulSet, DaemonSet, Configmap, secrets. When user creates the configuration from this template , this file will be embedded inside [`MustacheTemplate`](https://mustache.github.io/mustache.5.html). The `MustacheTemplate` is the template evaluation engine that is used to translate the template into final configuration file using the  custom parameters  and secret parameters passed by the user. So, it is advised to follow the `MustacheTemplate` syntax [`here`](https://mustache.github.io/mustache.5.html). The existing example of `deployment.yaml` is found [here](config-templates/redhat/ocs-local/4.6/deployment.yaml)


## Configuration of Single Valued variable
   This is simple variable with single value of type `text` , `boolean` or `number`. In the following example , `num-of-osd` , `ocs-upgrade`  and `billing-type` are the single valued variables. 

__Template YAML__
  ```
    - apiVersion: ocs.ibm.io/v1
      kind: OcsCluster
      metadata:
         name: {{ ocs-cluster-name }}
         namespace: default
      spec:
         monStorageClassName: localfile
         monSize: "1"
         osdStorageClassName: localblock
         osdSize: "1"
         numOfOsd: {{ num-of-osd }}
         billingType: "{{ billing-type }}"
         
  ```

__User Parameteter__

  ocs-cluster-name=mycluster
  num-of-osd=2
  billing-type=hurly


__Translated YAML__

```      
  - apiVersion: ocs.ibm.io/v1
    kind: OcsCluster
    metadata:
      name: mycluster
      namespace: default
    spec:
       monStorageClassName: localfile
       monSize: "1"
       osdStorageClassName: localblock
       osdSize: "1"
       numOfOsd: 2
       billingType: "hourly"

``` 

## Configuration of Multi Valued/List/Array/Csv variable

   If the type of the customer parameter is `csv` (Comma Separated Values), its considered as multi-valued variable. In the below example, `mon-device-path` is  the multi-valued variable. e.g  `mon-device-path=/dev/sdc1,/dev/sdc1` 

  __Template YAML__

   ```    
   - apiVersion: ocs.ibm.io/v1
    kind: OcsCluster
    metadata:
      name: {{ ocs-cluster-name }}
      namespace: default
    spec:
       monStorageClassName: localfile
       monSize: "1"
       osdStorageClassName: localblock
       osdSize: "1"
       monDevicePaths:
         {{#mon-device-path }}
         - "{{{ item }}}"
         {{/mon-device-path }}
   ```
  __Custom Parameters__

     mon-device-path=/dev/sdc1,/dev/sdc1
     ocs-cluster-name=mycluster

  __Translated YAML__  
 
   ```    
    - apiVersion: ocs.ibm.io/v1
      kind: OcsCluster
      metadata:
        name: mycluster
        namespace: default
      spec:
       monStorageClassName: localfile
       monSize: "1"
       osdStorageClassName: localblock
       osdSize: "1"
       monDevicePaths:
         - /dev/sdc1
         - /dev/sdc2
   ```

## Configuration for optional/empty/false  variable
   The optional user parameter may not be given by the user while creating the configuration. These variables will result into undefined or empty valued variables. If the variable is not defined or empty, its evaluated to empty string , that could break the YAML syntax. Hence it is advised to added `emptyness/undfined` check in the template as below.   

### Single value empty parameter
   If the parameter type is `text` , `number` or `boolean`  i.e not CSV (multi-valued), we use the `parameter name` in the condition as below  
   __syntax__
   ```
   {{#paramater-name}}
     yaml-tag
   {{/paramater-name}}
   ```

   The `yaml-tag` will be rendered only when parameter is not `empty` or `false`
   
   In this example, `monSize` is optional variable of `number` type. Hence, the  tag `monSize:`  is enclosed inside `{{#monSize}}`.
  
  __Template YAML__

  ```    
  - apiVersion: ocs.ibm.io/v1
    kind: OcsCluster
    metadata:
      name: {{ ocs-cluster-name }}
      namespace: default
    spec:
       monStorageClassName: localfile
       {{#monSize}}
       monSize: {{monSize}}
       {{/monSize}}`
       osdStorageClassName: localblock
       osdSize: "1"
   ```
  __Custom Parameters__

     ocs-cluster-name=mycluster

  __Translated YAML__  

   ```    
    - apiVersion: ocs.ibm.io/v1
      kind: OcsCluster
      metadata:
        name: mycluster
        namespace: default
      spec:
       monStorageClassName: localfile
       osdStorageClassName: localblock
       osdSize: "1"
   ```

### CSV type empty value parameter
   If the parameter tye is CSV (multi-valued), we use the `parameter name` followed by `?` in the condition as below
   __syntax__
   ```
   {{#paramater-name?}}
     yaml-tag
   {{/paramater-name?}}
   ```

   The `yaml-tag` will be rendered only when parameter is not empty

   In this example, `mon-device-path` is optional variable of `csv` type. Hence, the list tag `monDevicePaths:`  is enclosed inside `{{#mon-device-path?}}`.
  
  __Template YAML__

  ```    
  - apiVersion: ocs.ibm.io/v1
    kind: OcsCluster
    metadata:
      name: {{ ocs-cluster-name }}
      namespace: default
    spec:
       monStorageClassName: localfile
       monSize: "1"
       osdStorageClassName: localblock
       osdSize: "1"
       {{#mon-device-path?}}
       monDevicePaths:
       {{#mon-device-path?}}
         {{#mon-device-path }}
         - "{{{ item }}}"
         {{/mon-device-path }}
   ```
  __Custom Parameters__

     ocs-cluster-name=mycluster

  __Translated YAML__  

   ```    
    - apiVersion: ocs.ibm.io/v1
      kind: OcsCluster
      metadata:
        name: mycluster
        namespace: default
      spec:
       monStorageClassName: localfile
       monSize: "1"
       osdStorageClassName: localblock
       osdSize: "1"
   ```