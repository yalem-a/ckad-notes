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


