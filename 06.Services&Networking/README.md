# Services and Networking
## Services
Service is a kubernetes object that enables connectivity in k8 cluster
Types of services
- NodePort: 
  - makes internal pod accessible on a port on the node
  - allows connectivity towards pod in a node. Listens on a port on node and forward request on that port
  to a port on a pod running the application
- ClusterIP: 
  - creates virtual IP inside the cluster to enable communication between different services 
- LoadBalancer: 
  - creates a load balancer for our application to distribute traffic

### NodePort
* NodePort:workerNode:[service(clusterIP:port)]:TargetPort(on a pod)
* NodePort range: 30,000 - 32,767

- apply service:

```
kubectl create -f service.yml 
```
- check  service:
```
kubectl get svc
```
* when multiple pods exist NodePort by default uses the following algorithm to distribute traffic between the pods
- Algorithm: random
- sessionAffinity: Yes

### ClusterIP
Used for internal communication between components

### Ingress Networking
Ingress can do the following
  * Load balancing
  * Authentication
  * SSL and URL based routing configuration

  * * To expose ingress to outside world we still need NodePort or LoadBalancer services

How can we implement ingress?
1. Deploy reverse proxies: nginx, haproxy, traefik ==> This is called **_ingress controller_**
By default k8 cluster does not have ingress controller
2. specify set of rules ==> These set of rules are called **_ingress resources_**
These are available and can be created like any other k8 objects

**ingress controller**
Available controllers are : 
- GCP HTTPS controller - _supported by k8_
- nginx - _supported by k8_
- contour
- haproxy
- traefik
- Istio

**Implementing nginx controller**
- Prepare nginx-controller deployment as shown in the "yaml file"
- prepare configmap to be used by nginx controller, example, location of logs ,etc.
- Create a service to expose ingress controller to external world - The sevrice will be of type **NodePort** check "NodePort_service_for_nginx.yml"
- create a service account with correct roles and role bindings for nginx to have privilege to monitor k8 cluster. Check service account yml

**ingress resource**
- set of rules and configs applied on ingress controller
- Apply ingress
```
kubectl apply -f ingress.yml
```
- ingress Rules
  - rule-1
  - rule-2
  - ...
  - default rule (everything else)
check yaml files for different scenarios
- Checking ingress
```
kubectl describe ingress <ingress-name>
```

**Creating ingress imperatively**

```
kubectl create ingress <ingress-name> --rule="host/path=service:port"
```
Example:

```
kubectl create ingress ingress-test --rule="/wear=wear-service:8080" --rule="/watch=watch-service:8080" 
```
** Create a service to make it available to external users

```
kubectl expose deploy <deployment-name> -n <ns-name> --name ingress --port=80 --target-port=80 --type NodePort
```
To specify the node port edit the service using the following

```
kubectl edit svc ingress -n <ns-name>
```

### Network policies
- "Allow All": Kubernetes by default allows All traffic to any pod or service within the cluster
Network policy is a k8 object that allows implementing network security just like firewall rules.
- Network policy has one or more rules that either denies or allows a traffic
- Network policy can be applied to pods. 
- How do we apply network policy to a pod? This can be done using labels and selectors.

Example:
- Allow traffic to mysql pod on port 3306 from API-server 
Refer to _network-policy-definition.yml_ and _pod-with-network-policies.yml_ files for checking network polcies and how 
this is applied on the pod
- Apply network policy
```
kubectl apply -f network-policy-definition.yml
```
- check network policy
```
kubectl get netpol
```
- Network policies are implemented enforced using network solutions implemented on k8 cluster. 
- Examples of network solutions that support network policies:
  - kube-router
  - Calico
  - Romana
  - Weave-net
- Examples of network solutions that DO NOT support network policies:
  - flannel
#### Developing Network policies
- Requirements
  - Assume we have 3 pods: web-pod, api-pod, db-pod
  - We should only allow traffic from api-pod to db-pod on tcp port 3306
  - The rest can stay on default settings
Answer:
  - Check db-policy _network-policy-exercise_ directory
  - If we don't specify namespace in the network policy, it will be applied to all name-spaces. Meaning any pods in any 
  namespace with matching labels will be allowed to db-pod.
  - use case1: If we want to restrict access to a given namespace only, we use _namespaceSelector_ in addition is _podSelector_
  - use case2: If we don't specify _podSelector_ all pods within the specified namespace will reach out to db-pod
  - use case3: Let's assume there is backup server outside the namespace in which the db-pod exists, and we want 
  this backup server to reach db-pod. And this backup server is not deployed in k8 cluster, therefore our selectors 
  does not help(backup server is not a pod on a cluster). To solve we can refer to the IP address of the backup server.
  and this can be used in the network policy.
 **Rules are computed as below**
  - OR operator is used to compute _ipBlock_ and _nodeSelector_ 
  - AND operator is used to compute rules with _nodeSelector_ and _ipBlock_