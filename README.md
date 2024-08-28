# ckad

### _general_

```sh
kubectl run redis --image=redis --dry-run=client -o=yaml > redis.yaml
kubectl get pod redis -o yaml > edit.yaml
kubectl edit pod redis
kubectl replace --force -f edit.yaml

kubectl scale --replicas=6 -f deployment.yaml
kubectl scale replicaset my-replicaset --replicas=6

kubectl explain replicaset
kubectl get rs
kubectl delete pod xxx yyy zzz
```

### _deployment_
```sh
kubectl get all
kubectl create deployment hello --replicas=3 --image=nginx
kubectl create deployment hello --dry-run=client --replicas=3 --image=nginx -o yaml > sample.yaml
kubectl -o json, yaml, wide

kubectl get pods -A

kubectl create deployment hello --image=nginx --dry-run=client -o yaml > deployment.yaml

[Course: Kubernetes Certified Application Developer (CKAD) with Tests | Udemy](https://www.udemy.com/course/certified-kubernetes-application-developer/learn/lecture/14112621#reviews)
```

### _service_
```sh
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml

kubectl run --help
kubectl run nginx --image nginx --labels="tier=db"
kubectl create service --help
kubectl run httpd --image=httpd:alpine --dry-run=client -o yaml --expose=true --port 80
```

### _docker_
```sh
docker build -t webapp-color .
docker run --name webapp-color -p 8282:8080 webapp-color
docker run python:3.6 cat /etc/*release*
```

### _command and args_
```sh
kubectl replace
kubectl run webapp-green --image=nginx -- --color green
kubectl run webapp-green --image=nginx --command -- python app2.py -- --color green
```

### _configmap_
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
spec:
  containers:
  - env:
    - name: APP_COLOR
      valueFrom:
       configMapKeyRef:
         name: webapp-config-map
         key: APP_COLOR
    image: kodekloud/webapp-color
    name: webapp-color

### _secret_
kubectl create secret generic webapp-secret –from-literal=DB_HOST=mysql –from-literal=DB_PASSWORD=123
echo “mysql” | base64
Secret Store CSI Driver Tutorial | Kubernetes Secrets | AWS Secrets Manager | KodeKloud (youtube.com)

kubectl run xxx -- whoami

### _service account_
Kubectl create sa dashboard-sa
Create role and role binding
kubectl create token dashboard-sa

### _requests and limits_
image:
resources:
  requests:
    memory: “4Gi”
    cpu: 2
  limits:
    
With Requests, no limits for CPU
Requests = limits for memory

### _taint_
kubectl taint nodes node01 spray=mortein:NoSchedule
Kubectl taint nodes XXX app=blue:NoSchedule|PreferNoSchedule|NoExecute

### _toleration_
containers:
tolerations:
-	key: “app”
operator: “Equal”
value: “blue”
effect: “NoSchedule”

### _nodeSelector_
kubectl label nodes node01 size=Large
containers:
nodeSelector:
  size: Large

### _affinity_
containers:
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
         - key: size
           operator: In|NotIn|Exists
           values:
           - Large
           - Medium

kubectl label node node01 color=blue

kubectl describe node node01 | grep Taints

### _multi-container pods_
kubectl logs app -n elastic-stack
kubectl -n elastic-stack exec -it app – cat /log/app.log

### _readiness probes_
containers:
-	name: 
image:
ports:
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8

### _liveness probes_
containers:
-	name: 
image:
ports:
livenessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8

kubectl delete pods –all
kubect get pods -o yaml > pods.yaml

### _container logging_
kubectl logs -f <pod-name> <container-name>

### _metric server_
git clone https://github.com/kubernetes-incubator/metrics-server
kubectl create -f deploy/1.8+/

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git

kubectl top node
kubectl top pod

### _selectors_
kubectl get all –selector env=prod –no-headers | wc -l
kubectl get all –selector env=prod,bu=finance,tier=frontend

### _rolling updates and rollbacks_
kubectl create -f deployment.yaml --record
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment/my-deployment

kubectl set image deployment/my-deployment nginx-container=nginx:1.9.1

kubectl rollout undo deployment/my-deployment
kubectl rollout undo deployment/ my-deployment --to-revision=1

### _jobs_
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name:
          image:
      restartPolicy: Never

### _cronjobs_
spec:
  schedule: “* * * * *”
  jobTamplate:
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name:
          image:
      restartPolicy: Never

### _service - NodePort_
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: frontend

### _service - ClusterIP_
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    Port:80
  selector:
    app: myapp
    type: frontend

### _ingress controller_
kubectl create ingress ingress-pay -n critical-space –rule”/pay=pay-service:8282”

annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
  nginx.ingress.kubernetes.io/ssl-redirect: “false”

### _Network policy_
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: db-polciy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  Ingress
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306

### _volume_
spec:
  containers:
  - image:
    volumeMounts:
    - mountPath: / opt
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory

  volumes:
  - name: data-volume
    awsElasticBlockStore:
      volumeID: <volume-id>
      fdType: ext4

### _persistent volume_
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-poll
spec:
  accessModes:
    - ReadWriteOnce | ReadOnlyMany | ReadWriteMany
  capacity
    storage: 1Gi
  hostPath:
    path: /tmp/data

### _persistent volume claim_
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi

### _kubeconfig_
kubectl config view
kubectl config view –kubeconfig=my-custom-config
kubectl config use-context xxx
kubectl config -h
kubectl config use-context research --kubeconfig=/root/my-kube-config

### _roles_
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
-	apiGroups: [“”]
resources: [“pods”]
verbs: [“list”, “get”, “create”, “update”, “delete”]
resourceNamers: [“blue”, “orange”]

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
-	kind: User
name: dev-user
apiGroup: rbac.authorization.k8s.io
roleRef:
      kind: Role
      name: developer
      apiGroup: rbac.authorization.k8s.io

kubectl auth can-i create deployments
kubectl auth can-I delete nodes
kubectl auth can-I delete nodes –as dev-user –namespace test

kubectl describe pod kube-apiserver-controlplane -n kube-system | grep mode

kubectl config view

kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user

### _cluster roles_
kubectl api-resources – namespaced=true
kubectl api-resources – namespaced=false

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
-	apiGroups: [“”]
resources: [“nodes”]
verbs: [“list”, “get”, “create”, “update”, “delete”]

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
-	kind: User
name: cluster-admin
apiGroup: rbac.authorization.k8s.io
roleRef:
      kind: ClusterRole
      name: cluster administrator
      apiGroup: rbac.authorization.k8s.io

kubectl auth can-i list nodes --as michelle

### _admission controllers###
kube-apiserver -h | grep enable-admission-plugins
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins

kubectl exec -it kube-api-server-controlplane -n kube-system – kube-apiserver -h | grep ‘enable-admission-plugins’
/etc/Kubernetes/manifests/kube-apiserver.yaml
ps -ef | grep kube-apiserver | grep admission-plugins

kubectl create secret tls webhook-server-tls --cert /root/keys/webhook-server-tls.crt --key /root/keys/webhook-server-tls.key

### _api versions / deprecations_
kubectl api-resources

kubectl proxy 8001&
curl localhost:8001/apis/authorization.k8s.io

curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 | kubectl apply -f -

### _custom resource definition_
kubectl get crd
kubectl describe crd collectors.monitoring.controller

### _helm_
cat /etc/*release*
helm –help
helm env
helm version

helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo wordpress
helm repo list

helm install [release-name] [chart-name]
helm install release-1 bitnami/wordpress

helm list
helm uninstall my-release
helm pull –untar bitnami/wordpress
ls wordpress
helm install release-4 ./wordpress

helm repo list
