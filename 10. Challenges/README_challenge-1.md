# Challenge-1
## Roles and rolebindings
- Creating role and granting every access to pods,svc, pvc in the development namespace
```
kubectl create role developer-role --verb=* --resource=pods,svc,pvc -n development
```
- Creating rolebinding and assigning user _martin_ to the _developer-role_
```
kubectl create rolebinding developer-rolebinding --role=developer-role --user=martin -n development
```
## Security contexts
- Creating a context and assigning user _martin_ to context
```
kubectl config set-context developer --cluster=kubernetes --user=martin

```
- Creating user credentials for user _martin_ using private key and cert
```
kubectl config set-credentials martin --client-key=/root/martin.key --client-certificate=/root/martin.crt --embed-certs=false
```
## Creating pods
- Create a pod with initContainers having storage volume attached
```
apiVersion: v1
kind: Pod
metadata:
  name: jekyll
  namespace: development
  labels:
    run: jekyll
spec:
  containers:
  - name: jekyll
    image: kodekloud/jekyll-serve
    volumeMounts:
    - mountPath: /site
      name: site
  initContainers:
  - name: copy-jekyll-site
    image: kodekloud/jekyll
    command: ["jekyll"]
    args: ["new","/site"]
    volumeMounts:
      - mountPath: /site
        name: site
  volumes:
  - name: site
    persistentVolumeClaim:
      claimName: jekyll-site
```