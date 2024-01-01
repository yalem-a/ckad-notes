# Security
## Authentication, Authorization and Admission control
- Secure host
  - disable password authentication, only use ssh key based authentication
- Secure kubernetes (kube-apiserver)
Controlling access to kube-apiserver is the focus of this security module.
- Authentication: who can access the server?
  - username and password stored in static file
  - username and token stored in static file
  - certificate
  - LDAP
  - service accounts : this is for machines
- Authorization : what can they do?
  - RBAC
  - ABAC
  - Node authorizer
  - Webhook mode
### Authentication
- Admins and developers : User acconuts
- Bots - service accounts
** k8 does not manage user accounts natively; rathe relies on external source like file, certs, ldap.
We can not create user accounts directly on kubernetes, however,_Serviceaccounts_ can be created on kubernetes.
- Creating service account
```
kubectl create serviceaccount service1
```
- Checking service account
```
kubectl get serviceaccount
```
** All user acccess to k8 cluster is managed by _kube-apiserver_. 
- kube-apiserver does authentication in either of the following ways
  - static password file
  - static token file
  - Certificates
  - Identity services(LDAP)
- Basic authentication:
  - prepare csv file containing "password,username, userid,group" | "token,username, userid,group"
  - Pass the following to the kube-apiserver as an option (vi /etc/kubernetes/manifests/kube-apiserver.yaml)
    - --basic-auth-file=user-details.csv | --token-auth-file=user-token-details.csv
  - restart kube-apiserver once modification is made
- Checking user credentials from cli(username/password)
```
curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"
```
- Checking user tokens from cli(username/token)
```
curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer token"
```
_Notes_
- For maximum security consider using volume mount while providing auth file in kubeadm setup
- user RBAC
### Security KubeConfig
Below is the command used by client to query k8 restAPI for a list of pods
- Generate a certificate for a user (admin.key, admin.cert)
```
curl https://my-kube-playground:6443/api/v1/pods --key admin.key --cert admin.cert --cacert ca.crt
```
- This request will be sent to kube-apiserver for authnetication
** my-kube-playground - our cluster
- How can we do the above stuff using _kubectl_ command to query pods information
```
kubectl get pods 
                --server my-kube-playground:6443 
                --client-key admin.key 
                --client-certificate admin.crt 
                --certificate-authority ca.cert
```
- Running the above command is tedious, and we rather use _kubeconfig_ file to store this information 
- kubeconfig file is shown below
```
--server my-kube-playground:6443 
--client-key admin.key 
--client-certificate admin.crt 
--certificate-authority ca.cert
```
- Run the following command to utilize the kubeconfig file
```
kubectl get pods --kubeconfig config
```
  - By default, k8 is going to look for _config_ file in _$HOME./kube/config_
  - If kubeconfig file is located in _$HOME./kube/config_, we dont have to specify the path in the "kubectl get pod .." command
- Generally speaking kubeconfig file have the following sections
  - cluster : k8 cluster that the user has access to. Example Dev,production, Google, etc.
    - Server specification from the kubeconfig goes into cluster section : _--server my-kube-playground:6443_ 
    - it can be named as _MyKubePlayground_
  - users: user accounts with which you have accessed this cluster, Example: Admin. Dev User, Prod User 
    - user specifications from kubeconfig goes into users section: --client-key admin.key --client-certificate admin.crt --certificate-authority ca.cert
    - it can be named as _MyKubeAdmin_
  - context: Defines which user account will be used to access which cluster, Example: Admin@Production, Dev@Google
             _what use we are going to use to access what cluster_
             _This will allow us not to specify user certificate, server address in each and every kubectl command_
    - it can be named as _MyKubeAdmin@MyKubePlayground_
- Checking kubeconfig file. It also shows the current context
```
kubectl config view
```
By default, the above command  is going to use config file in $HOME/.kube/config.
- Checking kubeconfig file with custom config file
```
kubectl config view --kubeconfig=my-custom-config
```
- Changing context from _my-kube-admin@my-kube-playground_ to _prod-user@production_
```
kubectl config use-context prod-user@production
```
- Checking current context
```
kubectl config current-context
```
** Context section in kubeconfig file also has "namespace" which allows specifically swithcing to particular namespace
- Switching to another context based on non-default kubeconfig file
```
kubectl config use-context research --kubeconfig=/root/my-kube-config
```
- Make the my-kube-config file the default kubeconfig
  - move my-kube-config to config file $HOME/.kube/config
```
mv my-kube-config /root/.kube/config
```
### API Group
- Checking k8 version using api
```
curl https://kube-master:6443/version
```
- Checking pods using api
```
curl https://kube-master:6443/api/v1/pods
```
- Below are APIs in k8
- /health, metrics, /version, /api, /apis, /logs
- Below are APIs responsible for the cluster functionality
  - _Core group_ (core functionalities exist): /api
    - /v1 - namespaces, pods, pv, pvc, events, configMaps, secrets ,services, nodes, services, etc.
  - _Named group_(future updates will be under this): /apis
    - /apps, /extensions, /networking.k8s.io, /storage.k8s.io, /authentication.k8s.io, /certificates.k8s.io
- starting kube proxy
```
kubectl proxy
```
- Kube proxy vs kubectl proxy
  - Kube proxy: enables connectivity between pods and services accross nodes in the cluster
  - kubectl proxy: HTTP proxy service created by kubectl utility to access kube-apiserver
                   Rather than accessing the kube-apiserver directly kubectl proxy allows initiating traffic towards kube-apiserver

- API verbs: actions that can be done on resources
  - list, get, create, delete, update, watch
### Authorization
- Answers what can users do?
- Supported authorization mechanisms
  - Node authorization: handles the following authorization
  _user->accesses> kube-apiserver  <-accesses <-kubelet_
    - user accesses kube-apiserver: for managing the cluster   
    - kubelet reads (services, endpoints,Nodes, pods)
    - kubelet write (node status, pod status,events)
    - kubelet is part of system:node group
  - ABAC : dev user wants to access cluster view ,create and delete pod
    - permissions are directly attached to the users as shown below. 
      ```
      {"kind": "policy", "spec":{"user":"dev-user", "namespace":"*", "resource": "pods", "apiGroup":"*"} }
      ```
      - every time change is made to ABAC policy kube-apiserver must be restarted. This is makes it difficult
  - RBAC
    - Define a role-> associate permisions to the role->assign users that role
  - Webhook
    - outsources authorization mechanisms like open policy agent
    - authorization decisions are made externally
  - AlwaysAllow: always allows with checking authorizations (this is default)
  - AlwaysDeny: always denies with checking authorizations
- Authorization is configured in kube-apiserver _--authorization-mode=AlwaysAllow_
- If multiple authorization modes are set authorization is done sequentially one after the other
  - _--authorization-mode=Node,RBAC,webhook_
    - Node authorization is checked first, if it denies the request it goes to RBAC, if RBAC allows nomore checks are done
### RBAC
- Creating a role: use _Role_ object using yml file
```
kubectl create -f role.yml
```
- Check roles
```
kubectl get roles
```
- Assign a role to the user: use _RoleBinding_ object using yml file
```
kubectl create -f roleBinding.yml
```
- Check roleBindings
```
kubectl get rolebindings
```
- Check if me as a user have access to a particular resource in the cluster
```
kubectl auth can-i create deployments
```
```
kubectl auth can-i create node
```
- Impersonating user to check if the user has access to a particular resource in the cluster
```
kubectl auth can-i deployments --as dev-user
```
- Checking kube-apiserver settings
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
or
```
ps -aux | grep authorization
```
**Creating roles imperatively**
```
kubectl create role developer --verb=list,create,delete --resource=pods 
```
**Creating rolebinding imperatively**
```
kubectl create rolebinding dev-role-binding --role=developer --user=dev-user
```
### Cluster Role
Resources are generally categorized as _namespaced_ or _cluster_-scoped
- Roles and rolebindings are _namespaced_. Examples:
  - Examples: pods, replicasets,deployments, secrets, jobs, servies, roles , rolebindings, configmaps, pvcs
- Nodes are _cluster-scoped_ resources.
  - Examples: nodes, PVs, clusterroles, clusterrolebindings, certificatesigningrequests, namespaces

- Checking namespaced resources
```
kubectl api-resources --namespaced=true
```
- Checking cluster-scoped resources
```
kubectl api-resources --namespaced=false
```
- How do we authorize users to cluster-scoped resources?
  - Answer: We can use clusterroles and clusterrolebindings
  - Example: _clusterAdmin_ - with the following privileges
    - can view nodes, can create nodes, can delete nodes
            _storageAdmin_ - with the following privileges
    - can view PVs, can create PVs, can delete PVs

- Creating clusterroles 
```
kubectl apply -f clusterrole.yml
```
- Creating clusterrole imperatively
```
kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes
```
- Creating clusterrolesbinding - linking a user to the clusterrole
```
kubectl apply -f clusterrolebining.yml
```
- Creating clusterrolebinding imperatively
```
kubectl create clusterrolebinding node-reader-role-binding --clusterrole node-reader --user=michelle
```
- Checking if the user can now access the resources as per the roles given
```
kubectl get nodes --as michelle
```
**We can apply clusterrole to namesapced resources, that provides access these resources accross all namespaces in the cluster
- Defualt cluste roles
### Admission controllers
kubectl - authentication -> authorization -> admission controllers -> create pod
- Enforces better security measures on how the cluster is used
- Can change the request itself 
- Admission controllers go beyond authentication and authorization checks and do the following validations on yaml files
  - only allow images from certain registries
  - DO NOT permit "runAs root" user
  - Only permit certain capabilities 
  - Pods always have labels
- Examples of prebuilt admission controllers that comes with cluster by default
  - AlwaysPullImages: ensures images are always pulled when containers are created
    - DefaultStorageClass: observes the creation of PVCs and adds default storage class if not specified
    - EventRateLimit: limits the rate at which requests are coming into api-server
    - NamespaceExists: Rejects requests to namespaces that do not exist (deprecated)
    - NameSpaceAutoProvision (not enabled by default): automatically creates the namespace if it does not exist (deprecated)
    - NameSpaceLifecycle: replaces _NamespaceExists_ and _NameSpaceAutoProvision_

- Checking admission controllers enabled by default
```
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```
- To add admission controllers
  - in ADM setup
```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
add "- --enabe-admission-plugins=NodeRestriction"
```
  - kube-apiserver.service
```
add "--enabe-admission-plugins=NodeRestriction" for enabling
add "--disable-admission-plugins=NodeRestriction" for disabling

```

** Which admission controller is enabled in this cluster which is normally disabled?
Go to the following file and check _--enabe-admission-plugins_ section
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
- Checking enabled and disbaled admission plugins
```
ps -ef | grep kube-apiserver | grep admission-plugins
```
### Validating and mutating admission controllers

- _Validating admission controller_ (Example: _NamespaceExists_) : is a kind of admission controller tha checks if name space exists. Therefore, this referred 
to has _validating admission controller_. 
  - Validates the request and either allow or deny the request
- _Mutating admission controller_ (Example: _DefaultStorageClass_) : adds default storage class name in the PVC if it is not specified. This type of adimssion 
controller is known as mutating admission controller. 
  - It can change the object itself before it is created.
- There could be admission controllers that does both mutating and validation
  - Mutating admission controllers are invoked first before Validate admission controllers.
    - _NamespaceAutoProvision_(mutating) comes before _NamespaceExists_(validating)
*** We can also have our own admission controllers to support our own logic. To support external admission controllers 
    there are two admission controllers available.
    - MutatingAdmissionWebhook
    - ValidatingAdmissionWebhook
- This is implemented using an external AdmissionWebhook server that validates and/or mutates the request.
  - it receives a request in json format, and it responds with review outcome in json format.
- Steps to realize external admission controllers
  - step1: Deploy AdmissionWebhook Server
    - it is a basic api server which does _validate_ and _mutate_ implementations.
    - it can be deployed as deployment in k8 cluster
    - expose the deployment using k8 service (webhook service)
  - step2: configure webhook on k8 by creating webhook configuration object (check yml files)
    - configure k8 cluster to reach out to webhook to validate or mutate requests
Excercises:
  - Create TLS secret
```
kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key -n webhook-demo
```
### API versions
- Each API group has different versions
- An API group can support multiple versions at the same time
  - all these version can be used in the yaml files
- When an API group is at
  - /v1 - it means it is at ga stable version (generally available stable version)
    - enabled by default
    - it also a preferred storage version
  - /v1beta1: All major bugs in /v1aplha1 is fixed and end-to-end testing is done
    - enabled by default
    - might have bugs
  - /v1aplha1: an api is developed and merged to k8 code base, becomes k8 release for the 1st time
    - This is not enabled by default
    - this is not reliable
- If an object is created with API version set to v1alpha1 or v1beta1, those will be converted into preferred storage
  version which is v1 before storing them to etcd database
- Preferred version is a default version when we retrieve information through api, (like kubectl get ..)
- Storage version is a version in which an object is stored in etcd in respective of the version we used in yml files 
- Checking if preferred version of an API
  - Check the api url http://api-server/batch
- Checking if storage version of an API
  - Use ETCDCTL utility to query etcd database
- Enabling or disabling api versions is done in api server in "_--runtime-config_=batch/v2alpha"
### API Deprecations
- API group can support multiple versions at the same time. 
  - But why do we need to support multiple versions at the same time?
  - How many should we support?
  - When can we remove an older version when it is no longer required?
- All the above questions are answered by _API deprecation policy_

#### API Deprecation policies
- API elements may only be removed by incrementing the version of the API group
- API objects must be able to round-trip between API versions in a given release without information loss,
  except whole REST resources that do not exist in some versions
- There are also 2 or more  rules check the course pds
**kubectl convert**
- when a new k8 version is released new API groups are added old apis are deprecated and removed.
- We may have a lots of yml files in the old version
- To convert older yml files into a newer version we can use the following
```
kubectl convert -f <old-file> --output-version <new-api>
```
Example:
```
kubectl convert -f nginx --output-version apps/v1
```
** _kubectl convert_ command might not be installed by default
- Getting short names of API resources
```
kubectl api-resources
```
- Identify which API group a resource called _job_ is part of
```
kubectl explain job
```
- What is the preferred version for authorization.k8s.io api group?
step1: start a proxy to kube-apiserver
```
kubectl proxy 8001
```
step1: start a proxy to kube-apiserver
```
kubectl proxy 8001&
```
step2: run the following curl command
```
curl  http://localhost:8001/apis/authorization.k8s.io
```
* Enable the v1alpha1 version for rbac.authorization.k8s.io API group on the controlplane node.
 - step1: take a backup of existing manifest
 - step2: add the following
   * --runtime-config_=rbac.authorization.k8s.io/v2alpha

### Custom Resource Definitions
- create deploy
```
kubectl apply -f deploy.yml
```
- get deploy
```
kubectl get deploy
```
- delete deploy
```
kubectl delete -f deploy.yml
```
- All the above commands is going to create, list and modify deployment object/resource in ETCD datastore
- _Controller_: is going to determine the no. of pods that need to be created based on the _replica_ settings in deployment yaml file
  - Continuously monitors the status of resources that it is supposed to manage
- We can create object of mentioned in crd yaml file
```
kubectl create -f flight-ticket-definition.yml
```
- We can list the crd
```
kubectl get flightTicket
```
- We delete the crd
```
kubectl delete flightTicket
```
- we might also need a custom controller to make an API call to do booking on book-flight.com
** The above actions necessitates the availability of _custom resource_ and a  _custom controller_
#### Creating custom resource
- Before running above commands first we need to create CRD as below
```
kubectl create -f flight-ticket-custom-resource-definition.yml
```
- Then we can  the likes of the following
```
kubectl create -f flight-ticket-definition.yml
```
#### Creating custom controllers
Custom resource that been created in the above steps only stays in etcd database. If we need to watch these resources
we need a custom resource controller. actions on the crd is defined by custom controller.

**Exercises on CRD**

