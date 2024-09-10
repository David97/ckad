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
```sh
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard

apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
  - name: webapp-color
    image: kodekloud/webapp-color
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map
          key: APP_COLOR
```

### _secret_
```sh
kubectl create secret generic webapp-secret --from-literal=DB_HOST=mysql --from-literal=DB_PASSWORD=123
echo "mysql" | base64

Secret Store CSI Driver Tutorial | Kubernetes Secrets | AWS Secrets Manager | KodeKloud (youtube.com)

kubectl run xxx -- whoami
```

### _service account_
```sh
kubectl create sa dashboard-sa
Create role and role binding
kubectl create token dashboard-sa
```

### _requests and limits_
```sh
image:
resources:
  requests:
    memory: "4Gi"
    cpu: 2
  limits:
    
with requests, no limits for CPU
requests = limits for memory
```

### _taint_
```sh
kubectl taint nodes node01 spray=mortein:NoSchedule
kubectl taint nodes XXX app=blue:NoSchedule|PreferNoSchedule|NoExecute
```

### _toleration_
```sh
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

### _nodeSelector_
```sh
kubectl label nodes node01 size=Large

containers:
nodeSelector:
  size: Large
```

### _affinity_
```sh
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
```

### _multi-container pods_
```sh
kubectl logs app -n elastic-stack
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

### _readiness probes_
```sh
containers:
- name: 
  image:
  ports:
  readinessProbe:
    httpGet:
      path: /api/ready
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 8
```

### _liveness probes_
```sh
containers:
- name: 
  image:
  ports:
  livenessProbe:
    httpGet:
      path: /api/ready
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 8

kubectl delete pods --all
kubect get pods -o yaml > pods.yaml
```

### _container logging_
```sh
kubectl logs -f <pod-name> <container-name>
```

### _metric server_
```sh
git clone https://github.com/kubernetes-incubator/metrics-server
kubectl create -f deploy/1.8+/

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git

kubectl top node
kubectl top pod
```

### _selectors_
```sh
kubectl get all --selector env=prod --no-headers | wc -l
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

### _rolling updates and rollbacks_
```sh
kubectl create -f deployment.yaml --record
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment/my-deployment

kubectl set image deployment/my-deployment <container_name>=nginx:1.9.1

kubectl rollout undo deployment/my-deployment
kubectl rollout undo deployment/my-deployment --to-revision=1
```

### _jobs_
```sh
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
      - name:
        image:
      restartPolicy: Never
```

### _cronjobs_
```sh
spec:
  schedule: "* * * * *"
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
```

### _service - NodePort_
```sh
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: frontend
```

### _service - ClusterIP_
```sh
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    Port:80
  selector:
    app: myapp
    type: frontend
```

### _ingress controller_
```sh
kubectl create ingress ingress-pay -n critical-space --rule "/pay=pay-service:8282"

annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
  nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

### _Network policy_
```sh
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
```

### _volume_
```sh
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
```

### _persistent volume_
```sh
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-poll
spec:
  accessModes:
  - ReadWriteOnce | ReadOnlyMany | ReadWriteMany
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

### _persistent volume claim_
```sh
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
```

### _kubeconfig_
```sh
kubectl config view
kubectl config view --kubeconfig=my-custom-config
kubectl config use-context xxx
kubectl config -h
kubectl config use-context research --kubeconfig=/root/my-kube-config
```

### _roles_
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
  resourceNames: ["blue", "orange"]

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

kubectl auth can-i create deployments
kubectl auth can-i delete nodes
kubectl auth can-i delete nodes --as dev-user --namespace test

kubectl describe pod kube-apiserver-controlplane -n kube-system | grep mode

kubectl config view

kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
```

### _cluster roles_
```sh
kubectl api-resources -- namespaced=true
kubectl api-resources -- namespaced=false

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "update", "delete"]

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster administrator
  apiGroup: rbac.authorization.k8s.io

kubectl auth can-i list nodes --as michelle
```

### _admission controllers_
```sh
kube-apiserver -h | grep enable-admission-plugins
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins

kubectl exec -it kube-api-server-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'
/etc/Kubernetes/manifests/kube-apiserver.yaml
ps -ef | grep kube-apiserver | grep admission-plugins

kubectl create secret tls webhook-server-tls --cert /root/keys/webhook-server-tls.crt --key /root/keys/webhook-server-tls.key
```

### _api versions / deprecations_
```sh
kubectl api-resources

kubectl proxy 8001&
curl localhost:8001/apis/authorization.k8s.io

curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 | kubectl apply -f -
```

### _custom resource definition_
```sh
kubectl get crd
kubectl describe crd collectors.monitoring.controller
```

### _helm_
```sh
cat /etc/*release*
helm --help
helm env
helm version

helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo wordpress
helm repo list

helm install [release-name] [chart-name]
helm install release-1 bitnami/wordpress

helm list
helm uninstall my-release
helm pull --untar bitnami/wordpress
ls wordpress
helm install release-4 ./wordpress

helm repo list
```

### _challenge 1_
```sh
# pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jekyll-site
  namespace: development
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage

# role
kubectl create role developer-role --namespace=development --verb=* --resource=services,persistentvolumeclaims,pods

# rolebinding
kubectl create rolebinding developer-rolebinding --namespace=development --role=developer-role --user=martin

# user and context
nano ~/.kube/config

contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
- context:
    cluster: kubernetes
    user: martin
  name: developer

users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLVENDQWhHZ0F3SUJBZ0lJTVQ1S2RkbXdaeUV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6Q>
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBOEZnOWVpOUd6TDJKT1JaZ2dMa0wvWWNhcjVzbDU5V1FkRmREVkdkbnBGUDdTYnJECithUlkreTA2RkNWSk5BSkN1Yk81d>
- name: martin
  user:
    client-certificate: /root/martin.crt
    client-key: /root/martin.key

# context
kubectl config use-context developer

# pod
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: jekyll
  name: jekyll
  namespace: development
spec:
  initContainers:
  - name: copy-jekyll-site
    image: gcr.io/kodekloud/customimage/jekyll
    command: ["jekyll", "new", "/site"]
    volumeMounts:
        - name: site
          mountPath: /site
  containers:
  - image: gcr.io/kodekloud/customimage/jekyll-serve
    name: jekyll
    volumeMounts:
        - name: site
          mountPath: /site
    resources: {}
  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
  dnsPolicy: ClusterFirst
  restartPolicy: Always

# service
kubectl expose pod jekyll --port=8080 --target-port=4000 --name jekyll --type=NodePort --namespace development --dry-run=client -o yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: jekyll
  name: jekyll
  namespace: development
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 4000
    nodePort: 30097
  selector:
    run: jekyll
  type: NodePort
status:
  loadBalancer: {}
```

### _challenge 2_
```sh
# kubeconfig
nano ~/.kube/config

# apiserver
nano /etc/kubernetes/manifests/kube-apiserver.yaml

# core cdn deployment
kubectl edit deployments.apps -n kube-system

# node01
kubectl uncordon node01

# pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/web"

# pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi

# pod
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: gop-file-server
  name: gop-file-server
spec:
  containers:
  - image: kodekloud/fileserver
    name: gop-file-server
    resources: {}
    volumeMounts:
        - name: data-store
          mountPath: /web
  volumes:
    - name: data-store
      persistentVolumeClaim:
        claimName: data-pvc
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

# service
kubectl expose pod gop-file-server --port=8080 --target-port=8080 --name gop-fs-service --type=NodePort --namespace default --dry-run=client -o yaml > service.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: gop-file-server
  name: gop-fs-service
  namespace: default
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    nodePort: 31200
  selector:
    run: gop-file-server
  type: NodePort
status:
  loadBalancer: {}

# copy image
ls /media

scp /media/* node01:/web
```
### _challenge 3_
```sh
# vote deployment
kubectl create deployment vote-deployment --image kodekloud/examplevotingapp_vote:before -n vote --dry-run=client -o yaml > vote-deployment.yaml

# vote service
kubectl expose deployment vote-deployment --name vote-service -n vote --port 5000 --target-port 80 --type NodePort --dry-run=client -o yaml > vote-service.yaml

# redis deployment
kubectl create deployment redis-deployment -n vote --image=redis:alpine --dry-run=client -o yaml > redis-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: redis-deployment
  name: redis-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: redis-deployment
    spec:
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /data
          name: redis-data
        resources: {}
      volumes:
      - name: redis-data
        emptyDir:
          sizeLimit: 500Mi
status: {}

# redis service
kubectl expose deployment redis-deployment -n vote --port=6379 --target-port=6379 --type=ClusterIP --name=redis --dry-run=client -o yaml > redis-service.yaml

# db deployment
kubectl create deployment db-deployment -n vote --image=postgres:9.4 --dry-run=client -o yaml > db-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: db-deployment
  name: db-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: db-deployment
    spec:
      containers:
      - image: postgres:9.4
        name: postgres
        resources: {}
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-data
        env:
        - name: POSTGRES_HOST_AUTH_METHOD
          value: trust
      volumes:
      - name: db-data
        emptyDir:
          sizeLimit: 500Mi

# db service
kubectl expose deployment db-deployment -n vote --name db --port=5432 --target-port=5432 --type=ClusterIP --dry-run=client -o yaml > db-service.yaml

# result deployment
kubectl create deployment result-deployment --image kodekloud/examplevotingapp_result:before -n vote --dry-run=client -o yaml > result-deployment.yaml

# result service
kubectl expose deployment result-deployment --name result-service -n vote --port 5001 --target-port 80 --type NodePort --dry-run=client -o yaml > result-service.yaml
```

### _challenge 4_
```sh
# redis01
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis01
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /root/redis01

# redis cluster
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: "redis-cluster-service"
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      volumes:
      - name: conf
        configMap:
          name: redis-cluster-configmap
          defaultMode: 0755
      containers:
      - name: redis
        image: redis:5.0.1-alpine
        command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: 'status.podIP'
        ports:
        - containerPort: 6379
          name: client
        - name: gossip
          containerPort: 16379
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: '/data'
          readOnly: false
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi

```
