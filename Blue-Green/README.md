# Blue/Green Strategy

kubectl apply -k .  
kubectl apply -f wordpress-v5.yaml  
kubectl get all  
kubectl get po --show-labels -w  
minikube service wordpress  
kubectl describe service  wordpress  
kubectl patch service wordpress -p "{\"spec\":{\"selector\":{\"version\":\"v5.4.2\"}}}"  
kubectl describe service  wordpress  
kubectl delete deploy wordpress-v1  
kubectl get po --show-labels -w  
kubectl delete -f wordpress-v5.yaml  
kubectl delete -k .  
