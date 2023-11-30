# Kubernetes configuration
## building docker image (cd /into/the/directory where the Dockerfile exists run the following)
```
docker build -t ubuntu-sleeper .
```
##  COMMANDS AND ARGUMENTS

### Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file ubuntu-sleeper-2.yaml.

```
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "5000"]
```

Or 

```
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "5000"
```

### What command is run given the following the image is built from Dockerfile

- Answer: --color green (because, command in the pod definition takes precedence)
```
# Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```

```
#k8 pod
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```

### Create a pod with the given specifications. By default it displays a blue background. Set the given command line arguments to change it to green.
- pod Name: webapp-green
- Image: kodekloud/webapp-color
- Command line arguments: --color=green
```
kubectl run webapp-green --image=kodekloud/webapp-color -- --color greenpod/webapp-green
```

##  Environment variables
### setting environment variables in docker
```
docker run -e APP_COLOR=pink simple-webapp-color
```
### setting environment pod
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

### Ways of setting environmental variables
- plaintext
```
env:
  - name: APP_COLOR
    value: pink
```
- configMap
```
env:
  - name: APP_COLOR
    valueFrom:
       configMapKeyRef: 
```
- secrets
```
env:
  - name: APP_COLOR
    valueFrom:
       secretKeyRef: 
```
## configMaps
### Creating configMaps
Config maps can be created in the following different ways:
- imperative
syntax:
```
kubectl create configmap \
<configMap-name> --from-literal=<key>=<value>
```
OR
```
kubectl create configmap \
<configMap-name> --from-file=<path/to/the/file>
```

Example:
```
kubectl create configmap \
app-config --from-literal=APP_COLOR=pink
```
- declarative 
```
kubectl create -f config-map.yaml
```
### checking configMaps
```
kubectl get configamps
```
```
kubectl describe configamp app-config
```
### Referring configMap in pod definition file
- 3 ways
1. inject all from configMap 
```
envFrom:
   - configMapRef:
         name: app-config ## name of configMap
```
2. Inject single value from configMap
```
env:
   - name: APP_COLOR
     valueFrom:
        configMapKeyRef: 
          name: app-config
          key: APP_COLOR
```
3. Injecting configMap from volume

```
volumes:
- name: app-config-volume
  configMap:
    name: app-config
```
## secrets

### Creating secrets
secrets can be created in the following different ways:
- imperative
syntax:
```
kubectl create secret generic \
<secret-name> --from-literal=<key>=<value>
```
OR
```
kubectl create configmap \
<configMap-name> --from-file=<path/to/the/file>
```

Example:
```
kubectl create secret generic \
app-secret --from-literal=DB_HOST=mysql
```

- from file
```
kubectl create secret generic \
<secret-name> --from-file=<path/to/the/file>
```

- declarative 
```
kubectl create -f secret_definition.yml
```
### encoding and decoding secerts
- Encoding
```
echo -n 'mysql' | base64
```
- Decoding
```
echo -n 'Xdfdg==' | base64 --decode
```

### checking secrets
```
kubectl get secrets
```
```
kubectl describe secrets
```
### viewing encoded secrets
```
kubectl get secrets app-secret -o yaml
```
### Injecting secrets
- injecting as all env variables in to pods
```
---
spec:
  containers:
    - name: app_name
      image: app1
    ports:
      containerPort: 8080
    envFrom:
        - secretRef: 
            name: app-secret

```
- injecting as single env variables in to pod
```
---
spec:
  containers:
    - name: app_name
      image: app1
    ports:
      containerPort: 8080
    env:
      - name: DB_PASSWORD
        valueFrom: 
          secretKeyRef: 
            name: app-secret
            key: DB_PASSWORD
```
- injecting as volume
Eache secret key is a filename and the content is it is value
```
---
volumes:
      - name: app_name_secret_volume
        secret:
          secretName: app-secret
```
### Forcing pod edit
```
kubectl replace --force filename.yaml
```
## Docker security
- Full list of permission owned by root user can be found at
```
/usr/include/linux/capability.h
```
- Define non-root user in docker
```
docker --user=1000 run ubuntu sleep 3600
```
or 

```
FROM ubuntu

USER 1000
```

- Limit actions by docker user
```
docker run --cap-drop kill ubuntu

```
- Add permission to docker user
```
docker run --cap-add MAC_ADMIN ubuntu
```
- Run a container with all privileges (even root user on container does not have all privileges)
```
docker run --privileges  ubuntu
```
## Security context (kubernetes security)
- security can be applied on CONTAINER level or POD level
**At POD level**
```
# Security at the POD level
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  ....
```
**At CONTAINER level**
```
# Security at the POD level
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    securityContext:
    runAsUser: 1000
    capabilites:
    add: ["MAC_ADMIN"]
  ....
```
***NOTE*** 
capabilities are only supported at CONTAINER level not at POD level

## Service accounts
There are two types of accounts in k8
- user account : for an admin or developer to access k8 cluster
- service account : for a thirdparty application to acess k8 cluster , like prometheus, jenkins ,etc.
***Creating a service account***
```
kubectl create serviceaccount dashboard-sa
```
***view a service account***
```
kubectl get serviceaccount
```
***Note***
When a service account is created a token is also created which will be used by the application

***view a token associated with service account***
```
kubectl describe secret serviceaccountName-token-kbbdm
```
***Note***

Default tokens created are located in a location /var/run/secrets/kubernetes.io/serviceaccount

***Note***
In version 1.24, there is not token automatically generated when serviceaccount was created.
To create a token in Kubernetes 1.24
```
kubectl create token <serviceaccount-name>
```
The token will be shown in the screen and it is time bound

To create a non-expiry token

```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: my-secret
  annotations: kubernetes.io/service-account.name: dashboard-sa

```
## Resource requirements
- It is the kubernetes scheduler's job to find a node with sufficient resources to create the pod
- The defines in ints definition yaml file the resource it needs (memory, cpu)

***cpu settings***

- The lowest possible value is 0.001 or 1mili
- 1 CPU means 
  - 1 vCPU in AWS
  - 1 GCP core
  - 1 azure core
  - 1 Hyperthread
***memory settings***

- Ki stands for kibibyte
- Mi stands for mebibyte
- Gi stands for Gibibyte
**notes**
- 1Ki = 1,024 bytes
- 1Mi = 1,048,576 bytes
- 1Gi = 1,073,741,824 bytes

- By default, containers does not have limits on how much resources they consume on a given node. We can however add limits on the resource consumption
in the pod definition file
- Containers may attempt to use more resources than the predefined limits
   - in case of memory: the container can use a little more memory than limits, but if it continues to more memory, the container will terminated
and Out Of Memory (OOM)error is seen
   - in case of CPU: cpu usage is throttled to a predefined limits. It cant go beyond the limit.
- How do we ensure if pods have the default setting applied? This  can be done via LimitRange. This is applied on the namespace level.
This is enforced when a pod is created. Check the LimitRange definition file
- Is there a way where we can define a constraint on a node, to say that all pods in a given node could not consume more than this much of CPU and memory?
This can be done using resource quotas. This is done on a namespace level. Check the quota definition file

## Taints and Tolerations 
***Note*** 
- This topic tries to answer what pods are placed on which nodes?
- Taint is like a spray which avoids the bug
- The bug is intolerant to the spray 
- Some bugs can be tolerant to the spray, therefore can land on the person
- There are two things that can decide whether the bug can land on a person
  - The taint on the person
  - The bug's toleration level to that particular taint
- In the above example
   - The person is a NODE
   - The bug is POD
***Note***
   - Taints and tolerations decide what pods can be scheduled on node
   - By default pods does not have toleration
   - If we wanted a given pod to be placed on the node with taint, we can add toleration criteria to the pod
   - Taints are placed on nodes and tolertions are placed on pods
### Taints
- Adding a taint to the node
```
kubectl taint nodes <node-name> key=value:taint-effect
```
Example: 
```
kubectl taint nodes node1 app=blue:NoSchedule
```
- taint-effects : Determines what happens to pods that do not tolerate this taint
   - NoSchedule - The pod will not be placed on the node
   - PreferNoSchedule - The pod might  be placed on a node , but this is NOT guaranteed.
   - NoExecute - Pods will not be placed on the node and existing pods that do not tolerate will be evicted 
Tolerations : are placed on a pod

***Note*** 
- taints and tolerations does not tell the pod  to go to a particular node.
- Instead, it tells the node to only accept pods with certain tolerations.
- If our requirement is to restrict a pod to certain nodes, it is achieved through node affinity

***Note***
- During setup taint is set on the master node automatically that prevents any pods from being scheduled on this node.
- If it was not for this setting, master node could have hosted pods. To check this run the following command
```
kubectl describe node kubemaster |  grep Taint
```

VIP commands on noes
- Getting the nodes
```
kubectl get nodes
```
- Getting taints applied on a given node 
```
kubectl describe node node01 | grep Taint
```
- Create a taint on node01 
```
kubectl  taint nodes node01 spray=mortein:NoSchedule
```

- Command to remove a taint from controlplane

```
kubectl taint node <node-name> key=value:effect-
```
Example:
```
kubectl taint node controlplane node-role.kubernetes.io/master:NoSchedule-
```
### Node selectors
- This to force pods to a specific nodes
- We can use the following methods
   - node selectors
Label the nodes to use node selectors
```
kubectl label nodes <node-name> <label-key>=<label-value>
```
Example:
```
kubectl label nodes node-01 size=Large
```
Checking labels:
```
kubectl get nodes --show-labels
```

Now refer this label in the pod-definition file under "nodeSelector". check pod-definition file
   - Node affinity
Allows us to specify more complex requirments like below
** place a pod on a node that is labeled with "Large" or "Medium", Not on "Small".
This can not be achieved through "nodeSelectors"
### Node affinity
Node affinity features makes sure pods are hosted on particular nodes. Check the pod-definition file
What if node affinity does not match any nodes? This is answered node affinity types that is defined in the pod definition file

**Below are affinity types**
   - **required**DuringSchedulingIgnoredDuringExecution 
       _DuringScheduling_ (preferred)
          - Label is mandatory. If label is not found during the creation of the pod, creation of pod is ABORTED
       _DuringExecution_ (Ignored)
          - If label is removed once the pod is running nothing is going to happen. POD will continue to run

   - **preferred**DuringSchedulingIgnoredDuringExecution - Label is NOT mandatory. Try your best to match, if not found , ignore node affinity
      **- _DuringScheduling_ (preferred)**
          - Label is NOT mandatory. Try your best to match, if not found , ignore node affinity and create the pod
      **- _DuringExecution_ (Ignored)**
          - If label is removed once the pod is running nothing is going to happen. POD will continue to run
     - requiredDuringSchedulingRequiredDuringExecution (planned in the future releases)
      **- _DuringScheduling_ (preferred)**
          - Label is  mandatory. Try your best to match, if not found  creation of pod is ABORTED
      **- _DuringExecution_ (Required)**
            - If label is removed from a node once the pod is running the POD will be evicted/terminated from the node.
