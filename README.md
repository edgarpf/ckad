# CKAD

## Useful commands

```
kubectl config set-context --current --namespace=<insert-namespace-name-here>
kubectl apply -f file.yml

kubectl get po -A #get all the pods in all namespaces
kubectl get po pod_name
kubectl get po  --no-headers -o custom-columns=":status.phase" # mostrar a penas o valor da coluna status
# with wide you can see more details like which node the pod is running
kubectl get po -n namespace_name -o wide
kubectl get all
kubectl get po -o yaml > pod.yaml
kubectl get po,svc,deploy,rs,cm,ns,secret,node,job,cj,netpol,pv,pvc
kubectl get po --show-labels
kubectl get po -l label_name=value,label_name2=value2
kubectl get po --selector env=dev,bu=finance

kubectl delete po pod_name

kubectl scale rs rs_name --replicas=5
kubectl scale deploy deploy_name --replicas=5

kubectl describe po pod_name
kubectl describe cm # it will show all the cm of a namespace with their values
kubectl describe node node01 | grep -i taints # check if there is a taint in a node

#describe a secret shows the secrets base64 DECODED.
kubectl describe secret secret_name

kubectl run nginx --image=nginx -l=tier=db --port 7000 --expose
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yml

kubectl expose po nginx --name nginx-service --port 7000 --target-port 7000

kubectl edit po nginx

kubectl create deploy deploy_name --image=image_name --replicas=3
kubectl create ns namespace_name
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root

kubectl taint nodes node_name lavel=label_value:NoSchedule
kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule- # untaint the taint

kubectl label node node_name name_label=value

kubectl exec -it app -- cat /log/app.log -n elastic-stack
kubectl exec -it pod_name -- sh

kubectl logs pod_name
kubectl logs pod_name -c container_name # used when a pod has more than one container.

kubectl top node #show k8s metrics after metrics-server installation
kubectl top po

kubectl rollout status deploy nginx
kubectl rollout history deploy nginx
kubectl rollout undo deploy nginx

kubectl describe ingress -n name_namespace #to check default endpoint

kubectl config view
kubectl config use-config user@context

kubectl auth can-i create deploy --as dev -n namespace_name 

helm list -n nome_namespace
helm delete name_release -n name_namespace
helm repo list
helm search repo chart_name --versions
helm upgrade name_release name_chart -n name_namespace
helm install name_release chart_name --set key=value --set key2=value2 -n name_namespace
#show value than you can use in --set
helm show values bitnami/apache | yq e

#decode or encode a base64 text inside a file
base64 file_name -d > path/file
base64 file_name -e > path/file
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

## Job/CronJob

```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: neb-new-job
  namespace: neptune
spec:
  parallelism: 2
  completions: 3
  template:
    metadata:
      labels:
        id: awesome-job
    spec:
     containers:
     - name: neb-new-job-container
       image: busybox:1.31.0
       command: ["sleep",  "2", "&&", "echo", "done"]
     restartPolicy: Never
```

```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

## Service

```yml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort # or ClusterIP
  ports:
    - targetPort: 8080
      port: 8080
      nodePort: 30080 #not necessary in ClusterIP
  selector:
    name: simple-webapp
```

## Ingress

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
           name: pay-service
           port:
            number: 8282
```

## NetworkPolicy

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

## Volumes

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

```yml
apiVersion: v1
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

   volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```
## Liveness and Readiness

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```
