# Lighting bulbs
## Questions sets -1 

- Checking connectivity between two pods

```
nc -v -z -w 2 secure-service 80
```
- Checking pods with labels
```
kubectl get pods --show-labels
```

- changing deployment image
```
kubectl set image deploy nginx-deploy nginx=nginx:1.16.17
```
- changing rolling back to previous version
```
kubectl rollout undo deploy nginx-deploy
```
- Exposing deployment on a given port
```
kubectl expose deploy redis --port=6379 --name=messaging-serivce --namespace=marketing
```
- Exposing deployment using nodePort and edit the yaml file to include _nodePort_
```
kubectl expose deploy my-webapp --port=80  --type NodePort --dry-run=client -o yaml > svc.yml
```
- Creating ingress rule
```
kubectl create ingress my-ingress --rule="ckad-mock-exam.com/video*=my-video-service:8080"
```
- Tainting a node
```
kubectl taint node node01 app_type=alpha:NoSchedule
```