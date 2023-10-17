# useful tips
##  Create resources as required, instead of creating the files from scratch
```
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```
## Creating pod

```
kubectl run nginx --image=nginx 
```
## Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

## Create Deployment 
```
kubectl create deployment nginx --image=nginx
```
## Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```
## Generate Deployment with 4 Replicas
```
kubectl create deployment nginx --image=nginx --replicas=4 
```
## Scale deployment using kubectl scale
```
kubectl scale deployment nginx --replicas=2
```
## scale deployment by saving  YAML definition to a file and modify
Update the YAML file with the replicas or any other field before creating the deployment
```
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml  > nginx-deployment.yaml
```
## Service
### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```
OR

```
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```
### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
```
kubectl expose pod nginx --port=80 --name nginx-service type=NodePort --dry-run=client -o yaml
```
OR
```
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```