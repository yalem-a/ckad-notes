# Challenging lab questions

## Deploy a redis pod using the redis:alpine image with the labels set to tier=db.
- Answer
```
kubectl run redis --image=redis:alpine --labels="tier=db"
```
## Create a service redis-service to expose the redis application within the cluster on port 6379.
- Answer
```
kubectl expose pod redis --port=6379 --name=redis-service
```
## Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas.
- Answer
```
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```
## Create a new pod called custom-nginx using the nginx image and expose it on container port 8080
- Answer
```
kubectl run custom-nginx --image=nginx --port=8080
```
## Create a new namespace called dev-ns
- Answer
```
kubectl create ns dev-ns
```
## Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.
- Answer
```
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```
## Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd).The target port for the service should be 80
- Answer
```
kubectl run httpd --image=httpd:alpine --port=80 --expose=true
```