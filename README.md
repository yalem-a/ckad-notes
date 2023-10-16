# Useful CKAD notes
This README attempts to document important notes, as prepare for CKAD exam. I started my journey towards
CKAD exam on October 2023. I used @Mumshad Mannambeth CKAD course on udemy.

## Getting information regarding mandatory replicaset yaml fields and their corresponding contents
```
kubectl exaplain replicaset
```
## Basic contents of yaml for pod definition(nginx example)

```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
```
## Checking the node on which the pod exists
```
kubectl get pod -o wide 
```
## Generating yaml file from cli
```
kubectl run nginx --image nginx dry-run=client -o yaml
```
## ReplicaSet sample yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app
  labels:
    app: myapp
spec:
  template:
  
replicas:
selector:
     matchLabels:
        type: front-end
     
```
## Scaling replicas in ReplicaSet
### method - 1 : update contents of yaml file to the required number and run the following command
```
kubectl replace -f replicaSet-definition.yml
```
### method - 2 : Specify the no. of replicas and provide the yaml file name
```
kubectl scale --replicas=6 -f replicaSet-definition.yml
```
### method - 3 : Specify the type of object and the object name as follows
```
kubectl scale replicaset myapp-replicaset --replicas=6  
```
## Summary of commands for replicaSet
### Creating replica set
```
kubectl create -f eplicaSet-definition.yml
```
### Checking replica set
```
kubectl get replicaset
```
### Checking pods created as a result of the applied replicaset(the pod names take replicaset name as a prefix) 
```
kubectl get pod
```
### Deleting replica set [also deletes underlying pods]
```
kubectl delete replicaset <replicaset-name>
```
### update replica set 
```
kubectl replace -f replicaSet-definition.yml
```

### scale replica set without modifying the yaml file
```
kubectl scale --replicas=6 -f replicaSet-definition.yml
```
## Summary of commands for deployment
Fun fact :) 
- Deployments are used to roll-updates, rollback updates, apply updates all at once. (Big distinction from replicaSet)
- "ReplicaSet" and "Deployment" have almost the same content except for "kind" field
- Applying deployment will also create replicaset
### creating deployment
```
kubectl create -f deployment-definition.yml
```
### Checking deployment
```
kubectl get deployments
```
### Checking replica set created as a result of applying deployment
```
kubectl get replicaset
```
### Checking pods  created as a result of applying deployment (the pod names take deployment name as a prefix) 
```
kubectl get pods
```
## Summary of general commands
### Checking all objects 
```
kubectl get all
```