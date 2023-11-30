# Multi-container PODs
## Below are common patterns when designing multi-container pods
- **Ambassador** - Example: our application might need to connect to different db types depending on where we are interms of application delivery
     - We can connect to DEV DB for development 
     - We can connect to TEST DB for testing
     - We can connect to PROD for production
We can outsource this logic to a separate container within a pod so that our application refers to a db in localhost
and this container wil proxy this request to the right DB. This is known as **Ambassador** container.
- **Adapter** - Example: when different applications send different log formats to a centralized server. 
To convert the log format into common format we can use "Adapter" pattern
- **Sidecar** - Example: logging agent alongside webserver 
Examples of instances where multi-container pod is required
- web server and log agent 
***Notes***
- multi-container pod share the same lifecycle, network space, storage
## initContainers
Examples: 
  -  That is a task that will be run only one time when the pod is first created
  - A process that pulls a code or binary from a repository that will be used by the main web application.
  - When a POD is first created the initContainer is run, 
   and the process in the initContainer must run to a completion before the real container hosting the application starts.
**_Inspecting logs in containers in a multi-container pod_**
```
kubectl logs myapp-pod -c init-myservice
```