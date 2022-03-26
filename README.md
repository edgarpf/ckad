# CKAD

## Useful commands

```
kubectl apply -f file.yml

kubectl get po -A #get all the pods in all namespaces
kubectl get po pod_name
# with wide you can see more details like which node the pod is running
kubectl get po -n namespace_name -o wide
kubectl get all
kubectl get po -o yaml > pod.yaml
kubectl get po,svc,deploy,rs,cm,ns,secret,node
kubectl get po --show-labels
kubectl get po -l label_name=value,label_name2=value2

kubectl delete po pod_name

kubectl scale rs rs_name --replicas=5
kubectl scale deploy deploy_name --replicas=5

kubectl describe po pod_name
kubectl describe cm # it will show all the cm of a namespace with their values
kubectl describe node node01 | grep -i taints # check if there is a taint in a node

kubectl run nginx --image=nginx -l=tier=db --port 7000 --expose
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yml

kubectl expose po nginx --name nginx-service --port 7000 --target-port 7000

kubectl edit po nginx

kubectl create deploy deploy_name --image=image_name --replicas=3
kubectl create ns namespace_name
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root

kubectl taint nodes node_name spray=label_name:NoSchedule
kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule- # untaint the taint

kubectl label node node_name name_label=value

kubectl exec -it app -- cat /log/app.log -n elastic-stack
```

To access a service from another namespace use ***service_name.namespace_name.svc.cluster.local***

## Template labels:

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

## Command and args

```yml
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
    command: ["python", "app.py"]
    args: ["--color", "pink"]
```

## Env variables

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green
    image: kodekloud/webapp-color
    name: webapp-color
```

## ConfigMap

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - envFrom:
    - configMapRef:
         name: webapp-config-map
    image: kodekloud/webapp-color
    name: webapp-color
```

## Secret

```yml
apiVersion: v1 
kind: Pod 
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default 
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret
```

## Toleration

```yml
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: spray
    value: mortein
    effect: NoSchedule
    operator: Equal
```

## NodeAffinity

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In #Exists is a valid value here. In this case there will not be values 
                values:
                - blue
```

## Multi container pod

```yml
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - name: lemon
    image: busybox
    command:
      - sleep
      - "1000"

  - name: gold
    image: redis
```