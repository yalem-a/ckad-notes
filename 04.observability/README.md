# Observability 
## Readiness and liveness probes
### Readiness probes
- **pod status**: The state of the pod in its lifecycle. Below different pod states

**_Pending_**: scheduler to trying to find where to place the pod 

**_containerCreating_**: images are being pulled and container begin starting

**_running_** : once all the containers in a pod started it goes into running state until the program successfully completes or it is terminated
Use below command to check status of the pod
```
kubectl get pods
```

- **pod conditions**: 

- complements status of the pod
- it is an array of true / false values that tell us the state of the pod
Example: 
- PodScheduled - true /false
- initialized - true /false
- containersReady - true /false
- Ready - true /false [this is the value we are focused on].
When the pod is ready it means, the pod is ready to accept user traffic, however it does not always mean it is ready to accept user traffic
How do kubernetes know whether the application in the pod is ready to accept user traffic
To check pod conditions run the following command

```
kubectl describe pods <pod-name>
```

By default k8 assumes as soon as the container is created it is ready to accept traffic.
If the application within the container took long, the service still routes traffic to the container, because the container state is READY state

We need to tie the ready condition to the actual state of the application in a container.

There are different ways to define if the application in a container is in a ready state using READINESS probes.

- HTTP Test - in 
- TCP test on db port, 3306, 
- Exec custom script to check application status

How readiness probes are used in multi pod setup?
- When we have multi pods in the replicaSet and when we add more pod, the service start to route traffic to newly created pod unless we have readinessProbe setup.
This will allow us ensuring the traffic will go the new pod only when the readinessProbe checks are successful.
### liveness probes
If the container is fine and the pod is fine, but the users are having issues accessing the application due to the issues in the application.
In this case we need **liveness probe** to check the application with the container is healthy.
If the container is unhealthy the container will be destroyed and re-created again
