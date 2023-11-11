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


