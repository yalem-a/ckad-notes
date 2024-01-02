# Helm
## Helm introduction
- A typical WordPress app requires the following in k8
  - Deployment, service, pv, pvc,secret
  - Every object needs a separate yaml file
- Helm - package manager for k8
  - Treats each application as single entity
  - It helps manage objects underneath the applications
  - Example: Installing WordPress application just takes the following command
  ```
  helm install wordpress ...
  ```
- Helm captures values of various objects in _values.yml_ fil
- We can upgrade with a single command
  ```
  helm upgrade wordpress ...
  ```
- We can roll back to previous version with a single command
  ```
  helm rollback wordpress ...
  ```
- We can un-install  a single command
  ```
  helm uninstall wordpress ...
  ```
## Installing Helm
- Prerequisite for installing helm
  - working k8 cluster
  - kubectl utility
  - valid credentials
- can be installed on linux as follows
```
sudo snap install helm --classic
```
## Helm concepts
- Step:1: Create templates example: templates/deployment
  - In each yaml file, for the variables whose values might change at run time we can parameterize them as follows
```
image: {{ .Values.image  }}
```
- Step:2: Create templates example: templates/deployment - check the yaml files
- _templates + values = Charts_ (definition files for deploying the application)
- Information about the chart is self is located in charts.yml
- We can navigate to the following url to explore existing charts
```
https://artifacthub.io/
```
- Or we can use the following helm command
```
helm search hub wordpress
```
- adding repositories in helm
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
- We can then search for repos in the bitnami
```
helm search repo wordpress
```
- We list existing repos
```
helm repo list
```
- Installing a chart (each installation creates a release ) - download and installs the packages
```
helm install <release-name> <chartname>
```
- If we just need to download the charts and unzip it
```
helm pull --untar bitnami/wordpress
```
- Then we can manually install downloaded charts
```
helm install release-4 ./wordpress
```
- List installed packages
```
helm list 
```
- Un-install packages 
```
helm uninstall <release-name> 
```