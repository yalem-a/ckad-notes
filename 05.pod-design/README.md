# POD Design 
## Labels, selectors , annotations
Labels are differentiators. Example: Mammals, birds, etc//
Selectors are filters. Filters based on type, based on functionality , etc.  

- Checking pods based on the selectors 
```
kubectl get pod --selector app=app1
```

-Creating a replica set consisting of 3 different kinds of pod refer <replicaset-definition.yml> file
we can use selectors for replicaSet to discover the pods
- Labels and selectors used to group objects
### annotations
Annotations are used to record detail for informational purposes.
Examples of annotations:
- name, version, email , etc. 

How many pods exist in **dev** environment? There are many pods 

```
kubectl get pod --selector env=dev
```

To remove the header

```
kubectl get pod --selector env=dev --no-headers
```

To get the exact count 

```
kubectl get pod --selector env=dev --no-headers | wc -l
```

*** How many objects are in the prod environment 
including PODs, ReplicaSets and any other objects?

```
kubectl get all  --selector env=prod --no-headers | wc -l
```
*** Identify the POD which is part of the prod environment,
the finance BU and of frontend tier?

```
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

## Updates and rollbacks
### rollout and versioning

- Create deployment -> triggers rollout -> new rollout > new deployment revision - > revision-1
- when a container version is updated > new rollout -> new deployment revision > revision-2
- This will allow us keep track of the changes made to our deployment and enables us rollback to previous version of 
deployment if necessary

- Checking status of rollout status
```
kubectl rollout status deployment/my-deployment
```

- Checking revision and history of rollout
```
kubectl rollout history deployment/my-deployment
```

### Deployment strategy
Two types deployment strategy
  - **_Recreate strategy_** - Destroy existing instances all at once  and create new instances - **There is downtime**
  - **_Rolling update_** - we don't destroy all instances at once, instead we take down the older version and bring up
a newer version _one by one_. The application is not down during this team and the upgrade is seamless. **This default strategy**

Possible changes in the application can be 
- image version change
- no. of replicas change
- selectors change

How can we apply changes in the application

1. run the following command 

```
kubectl apply -f deployment-definition-upgrade-strategy.yml
```
This will trigger new rollout and new revision of the deployment is created
2. run the following command 
*** This is if we are changing only the image version(This deployment.yml file will be unchanged!!)
```
kubectl set image deployment/myapp-deployment nginx-controller=nginx:1.9.1
```
Checking deployment strategy
```
kubectl describe deployment myapp-deployment
```
***Focus on the **StrategyType** and **Events** section of the output

How does the deployment performs an upgrade

- When a new deployment is created - > creates replicaset auotmatically -> creates no. of pods specified
- When we upgrade a deployment -> new replicaset is created following rolling update strategy
- We can check this using the following command
```
kubectl get  replicasets
```
### Rollback
k8 deployment will allow us rollback to previous revision if something is wrong in the current revision
- To undo a change run the following command
```
kubectl rollout undo deployment/myapp-deployment
```
This will destroy the current pods and brings back pods in the older revision

### Additional deployment strategies
-- These strategies can not be specified in the deployment file, but can be applied in a different way
- **Blue/Green** : The new version is deployed alongside the old version. 
  - The old version is called **Blue**
  - The new version is **Green**
** _All traffic is routed to old version(blue) and tests are done on Green version. Once all the tests passed we route 
  traffic to newer version (Green) using k8 service configuration
  
--> this is implemented in service mesh called istio
- **Canary**: Deploy a new version of the application and only route small percentage of the traffic to it
Majority of the traffic is still routed to the older version; only small percentage is sent to newer version.
If everything is okay, we update all the older version pods to the newer version
  - Deploy the new version in a new deployment. 
  - How can we achieve the following to realize canary deployment?
    - route traffic to both versions of the deployment?
      - create a common label for pods in both deployments, update the selector in service to match the new label
      applied on both deployments. [this achieves is 50/50% traffic distribution to both deployments]
    - route small percentage of traffic to newer version?
      - reduce the no. of pods in the new version of the deployment to minimum possible which is 1. Controlling the exact 
      percentage of traffic to be routed to newer version is impossible in the k8 native implementation. 
      istio comes to risk us in this case
Once tests are completed on the newer version of the deployment , update the older version of the deployment and delete
      the newer version.
Istio allows managing both blue/green and canary deployments very well. Kubernetes native implementation utilizes 
deployment and service implementation. 
## Jobs
Workload types a container can serve: web, application, db - this workloads meant to run for a long time
batch processing , analytics - run for a while and finish - meant to live for a short a period of time

Examples:
- calculate 3+2 
```
kubectl run ubuntu expr 3 + 2
```
- The above expression can be run in kubernetes using pod, check "pod-definition-math-expression.yml"
- Apply pod definition. In this the pod will compute 3 + 2 and the state will be changed to "completed" and
will restart to re-compute. this will continue until a threshold is reached

*** This is due to the fact that default restart policy which is "Always"
```
kubectl apply -f jobs-pod-definition-math-expression.yml
```
We need a manager that assigns jobs to a pods and makes sure the work is done successfully 
_replicaset_: ensures a given no. of pods are always running a given time
_Job_: is used to run a set of pods to perform a given task to completion. a job can be created using a definition file.

- Creating jobs
```
kubectl apply -f jobs-definition-math-expression.yml
```
- checking jobs and associated jobs
```
kubectl get jobs
```
```
kubectl get pod
```
- checking output of the job
```
kubectl logs <pod-name>
```
- deleting a job - deleting job also deletes associated pods
```
kubectl delete job <job-name>
```
### Creating multiple pods for jobs
refer to "jobs-multiple-pod..." definition file
according to this , pods are created one after the other. To change this behaviour use the "parallelism" field in the
job definition file

## CronJobs
A **Cronjob** is a job that can be scheduled.

"*/1 * * * *" -> minute hour dayOfTheMonth Month dayOfTheWeek

- creating cronJobs
```
kubectl create -f cronjob.yml 
```
- checking  cronJobs
```
kubectl get cronjob 
```