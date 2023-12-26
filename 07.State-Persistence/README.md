# State Persistence
## Volumes
**Storage is defined with in the pod definition file by each user creating the pod.
- Refer to yml files 
- Below are volume storage options
  - hostPath - uses directory from host system. Can not be shared between various nodes on the cluster

## persistent Volumes (PV)
**Storage centrally by administrator and users will consume this .
- _Persistent volume_ is a system-wide pool of storage volumes configured by administrator to be used by users deploying 
applications on the cluster.
- The users can now select storage from this pool using persistent volume claim (PVC)
- Access modes: 
  - Defines how a volume should be mounted on the host
    - ReadOnlyMany
    - ReadWriteOnce
    - ReadWriteMany
- Creating persistent volume
```
kubectl apply -f pv.yml
```
- Checking persistent volume
```
kubectl get pv
```
## persistent Volume claims (PVC)
once the PVs are created by administrator, any user can make the request(_claim_) using PVC definitions to
make the storage available to the node
- Once PVC are created, k8 binds pvc to PV based on the request and properties set on the volume
- Every PVC is bound to a single PV based on the following criteria
  - sufficient capacity
  - access modes
  - volume modes
  - storage class and 
  - selector (this is used if there are multiple possible matches)
- Smaller claim might bound to larger volume if all the other criteria matches and there are no better options
- there is 1 to 1 relationship between claims and volumes. No other claims can utilize capacity in the volume

- Creating persistent volume claim
```
kubectl apply -f pvc.yml
```
- Checking persistent volume claim
```
kubectl get pvc
```
- Deleting persistent volume claim
```
kubectl delete pvc <pvc-name>
```
** After deleting the claim what happens to the underlying pv?
This can be decided based on _PersistentVolumeReclaimPolicy_ property. Values include:
* Retain (default) - The persistent volume will remain after pvc is deleted. It will not be available for other claims
* Delete - As soon as the claim is deleted the claim will also be deleted.
* Recycle - The data in the volume will be scrapped before making it available to other claims
## storage classes (optional)
- _static provisioning:_ provision disks manually -> create pv -> create pvc
- It would have been great if the disks are provisioned automatically when the application requires it.
  - This is achieved via _storageClasses_
- _Dynamic provisioning_: Define provisioner such as google storage that can automatically provision storage on Google
cloud and attach that to pods when a claim is made. This is done by utilizing storageClasses. Refer to yml file.

** Introduction of storageClass avoids the necessity to manually create PV definitions.
** When a claim is made , the provisioner defined in the storageclass will create PV and assigns it to the PVC.

- Creating storage classes
```
kubectl apply -f sc.yml
```
- Checking persistent volume claim
```
kubectl get sc
```
- Deleting persistent volume claim
```
kubectl delete sc <pvc-name>
```
## stateful sets(optional)
- stateful sets are similar to deployment sets as in they create pods based on template , they can scale up, scale down.
- Here are differences between deployment and stateful 
  - in a sateful sets pods are created sequentially.
  - statefulsets assign a unique ordinal index to each pod, starting from 0,1,2,..n. Therefore; each pod gets a unique 
  name consisting of a unique index and statefullset name

- Creating stateful set 
```
kubectl apply -f state.yml
```
- Scaling stateful set 
```
kubectl scale statefulset mysql --replicas=5
```
- Checking stateful set 
```
kubectl get statefulset
```
- Deleting stateful set 
```
kubectl delete statefulset <setname>
```
## headless service
- Assume we have deployed mysql db using statefulset. The master need to serve write and read requests.
while the slaves should only serve ready requests. How can we direct _write_ requests to only master db?
We can use _headless_ service
- Headless service has the following characteristics
  - it is created like a normal service, but does not have IP
  - It does not perform load balancing
    - All it does is create dns entry for each pods using a pod name and sub-domain
      DNS entry patter looks like : _podName.headless_sevriceName.namespace.svc.clusterDomain_
      Example: mysql-0.mysql-h.default.svc.cluster.local
               mysql-1.mysql-h.default.svc.cluster.local
               mysql-2.mysql-h.default.svc.cluster.local
  - Now any service wanting to send write request to master db, can use mysql-0.mysql-h.default.svc.cluster.local

Check headless.yml to see how headless is defined.

- creating headless stateful set 
```
kubectl apply -f headless.yml
```
** When headless service is created DNS entries are created for PODs only if 2 conditions are met while creating the pod
  - under spec sections we have two optional fields
    - _hostname_: the name to be referred in the DNS name: _Example_: mysql-pod  
    - _subdomain_ - should be the same as the name of headless service: _Example_: mysql-h
     Then the DNS entry will be 
      mysql-pod.mysql-h.default.svc.cluster.local
    ** if hostname and sub-domain is not given in the pod , by default headless service does not create A-record for the 
       pod.
## storage in a statefulset 
- if we want to have a separate pvc for each pods in the deployment or statefulset, we have to use _volumeClaimTemplate_
within the statefulset  manifest. In this case when the pods are created sequentially, pvc is also done sequentially.