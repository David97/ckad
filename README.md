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
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
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
    - mountPath: /opt
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
  storageClassName: ""
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
  storageClassName: ""
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
  selector:
    matchLabels:
      app: redis-cluster # has to match .spec.template.metadata.labels
  serviceName: "redis-cluster-service"
  replicas: 6 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: redis-cluster  # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
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
              fieldPath: status.podIP
              apiVersion: v1
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi

# redis cluster service
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
spec:
  type: ClusterIP
  ports:
    - name: client
      port: 6379
      targetPort: 6379
    - name: gossip
      port: 16379
      targetPort: 16379
  selector:
    app: redis-cluster

# redis cluster config
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 {end}')
```

### _tips_
```sh
KUBE_EDITOR=nano kubectl edit deploy nginx

alias k=kubectl

k config set-context mycontext --namespace=mynamespace

kubectl explain cronjob.spec.jobTemplate --recursive

kubectl expose

--dry=run=client -o yaml

kubectl run nginx --image=nginx (deployment)
kubectl run nginx --image=nginx --restart=Never (pod)
kubectl run nginx --image=nginx --restart=OnFailure (job)
kubectl run nginx --image=nginx --restart=OnFailure --schedule="* * * * *" (cronJob)

kubectl run nginx --image=nginx --restart=Never --port=80 --namespace=myname --command --serviceaccount=mysa1 --env=HOSTNAME=local --labels=bu=finance,env=dev --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' --dry-run -o yaml - /bin/sh -c 'echo hello world'

kubectl run frontend --replicas=2 --labels=run=load-balancer-example --image=busybox --port=8080
kubectl expose deployment frontend --type=NodePort --name=frontend-service --port=6262 --target-port=8080
kubectl set serviceaccount deployment frontend myuser
kubectl create service clusterip my-cs --tcp=5678:8080 --dry-run -o yaml
```

### _lab 1_
#### Q1
```sh
# pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  hostPath:
    path: /opt/volume/nginx
  storageClassName: "manual"

# pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: "manual"

# pod
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: logger
  name: logger
spec:
  containers:
  - image: nginx:alpine
    name: logger
    volumeMounts:
    - name: log
      mountPath: /var/www/nginx
    resources: {}
  volumes:
      - name: log
        persistentVolumeClaim:
          claimName: log-claim
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

#### Q2
```sh
kubectl exec -it webapp-color -- sh
nc -v -z -w 2 secure-service 80
```
Incoming or outgoing connections are not working because of network policy. In the default namespace, we deployed a default-deny network policy which is interrupting the incoming or outgoing connections.

Now, create a network policy called test-network-policy to allow the connections, as follows:-
```sh
kubectl get pod --show-labels
```

```sh
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

then check the connectivity from the webapp-color pod to the secure-pod:-

```sh
root@controlplane:~$ kubectl exec -it webapp-color -- sh
/opt # nc -v -z -w 5 secure-service 80
```

#### Q3
Create a namespace called dvl1987 by using the below command:-

```sh  
$ kubectl create namespace dvl1987
```

Solution manifest file to create a configMap called time-config in the given namespace as follows:-

```sh
apiVersion: v1
data:
  TIME_FREQ: "10"
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987
```

Now, create a pod called time-check in the same namespace as follows:-

```sh
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  volumes:
  - name: log-volume
    emptyDir: {}
  containers:
  - image: busybox
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
    volumeMounts:
    - mountPath: /opt/time
      name: log-volume
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log"
```

#### Q4
Run the following command to create a manifest for deployment nginx-deploy and save it into a file:-

```sh
kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=4 --dry-run=client -oyaml > nginx-deploy.yaml
```

and add the strategy field under the spec section as follows:-

```sh
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
```

So final manifest file for deployment called nginx-deploy should looks like below:-

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx
```

then run the kubectl apply -f nginx-deploy.yaml to create a deployment resource.

Now, upgrade the deployment's image version using the kubectl set image command:-

```sh
kubectl set image deployment nginx-deploy nginx=nginx:1.17
```

Run the kubectl rollout command to undo the update and go back to the previous version:-

```sh
kubectl rollout undo deployment nginx-deploy
```

#### Q5

Solution manifest file to create a deployment redis as follows:-
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: redis-config
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: redis-config
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
```

### _lab 2_
#### Q1
The pod nginx1401 is not in a Ready state as the Readiness Probe has failed. Here is the solution YAML file:
```sh
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx1401
  namespace: dev1401
spec:
  containers:
  - image: kodekloud/nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /
        port: 9080
    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
```

#### Q2
```sh
kubectl create cronjob dice --image=kodekloud/throw-dice --schedule='*/1 * * * *' --dry-run=client -o yaml > cronjob.yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25 # This is so the job does not quit before it succeeds.
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: dice
            image: kodekloud/throw-dice
          restartPolicy: Never
```

#### Q3
```sh
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-busybox
  name: my-busybox
  namespace: dev2406
spec:
  containers:
  - image: busybox
    name: secret
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: /etc/secret-volume
      name: secret-volume
      readOnly: true
    resources: {}
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
  nodeSelector:
    kubernetes.io/hostname: controlplane
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

#### Q4
```sh
kubectl create ingress ingress-vh-routing --rule "watch.ecom-store.com/video=video-service:8080" --rule "appareals.ecom-store.com/wear=apparels-service:8080" --dry-run=client -o yaml > ingress.yaml

kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/video"
        backend:
          service:
            name: video-service
            port:
              number: 8080
  - host: apparels.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/wear"
        backend:
          service:
            name: apparels-service
            port:
              number: 8080
```

#### Q5
```sh
kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt
```

### _Mock Exam 2_
#### Q4
```sh
kubectl create ingress ingress --rule="ckad-mock-exam-solution.com/video*=my-video-service:8080" --dry-run=client -o yaml > ingress.yaml
```

#### Q7
```sh
kubectl create job whalesay --image=docker/whalesay --dry-run=client -o yaml > whalesay.yaml
```
