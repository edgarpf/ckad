# CKAD
```
kubectl apply -f file.yml

kubectl get ns
kubectl get po
kubectl get po pod_name
kubectl get po -n namespace_name -o wide # with wide you can see more details like which node the pod is running
kubectl get all
kubectl get po,svc,deploy,rs

kubectl delete po pod_name

kubectl describe po pod_name

kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yml
kubectl edit po nginx

kubectl get po --show-labels
kubectl get po -l label_name=value,label_name2=value2
```

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        tier: front-end # must be equal to the above
     spec:
       containers:
       - name: nginx
         image: nginx 
```