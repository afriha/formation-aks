# Namspace
To deploy a namespace, you need to create a YAML file named **namespace.yaml**. Use the example below:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
```bash
kubectl apply -f filename.yaml
kubectl get namespaces
kubectl get namespace dev
```
## ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: compute-quota
    namespace: dev
spec:
    hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: 5Gi
        limits.cpu: "10"
        limits.memory: 10Gi
```
```bash
kubectl apply -f filename.yaml
kubectl get resourcequota -n dev
```
# Pods
To deploy a POD, you need to create a YAML file named **pod.yaml**. Use the example below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
    env: production
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
```
Then run kubectl apply command against this file.
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev
```
You should see an error about defining resource limits. Jump to next section
## Resource Limits
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
    env: production
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
    resources:
      requests: {cpu: "250m", memory: "64Mi"}
      limits:   {cpu: "500m", memory: "128Mi"}
    livenessProbe:
      httpGet: {path: /, port: 80}
      initialDelaySeconds: 10
```
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev
# Cleanup
kubectl delete -f filename.yaml
```
# Manual Scheduling
## Node selector
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 namespace: dev
spec:
  containers:
  - name: nginx-container
    image: nginx
  nodeName: node01
```
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev
# Cleanup
kubectl delete -f filename.yaml
```
## Taint and tolerations
Add a taint to your node
```bash
kubectl taint nodes node01 app=blue:NoSchedule
```
Apply a toleration to your pod
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod-taint
 namespace: dev
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev
# Cleanup
kubectl delete -f filename.yaml
```
## NodeSelector
```bash
kubectl label nodes node01 size=small
```
Apply the nodeSelector to your pod
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod-selector
 namespace: dev
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
  nodeSelector:
    size: small
```
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev
# Cleanup
kubectl delete -f filename.yaml
```
## NodeAffinity
```yaml
apiVersion:
kind: Pod
metadata:
 name: myapp-pod-affinity
 namespace: dev
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values: [small]
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev
# Cleanup
kubectl delete -f filename.yaml
```
# Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.24
        name: nginx
        ports:
        - containerPort: 80
        resources:
          requests: {cpu: "250m", memory: "64Mi"}
          limits:   {cpu: "500m", memory: "128Mi"}
```
```bash
kubectl apply -f filename.yaml
kubectl get pods -n dev

# On change la version d'image:
kubectl set image deployment/nginx-deploy nginx=nginx:1.25 -n dev

# Observer en temps réel :
kubectl get pods -n dev -w

# Check history
kubectl rollout status deploy/nginx-deploy -n dev
kubectl rollout history deploy/nginx-deploy -n dev

# Rollback
kubectl rollout undo deploy/nginx-deploy -n dev
# Rollback vers révision spécifique :
kubectl rollout undo deploy/nginx-deploy --to-revision=1 -n dev

# Cleanup
kubectl delete -f filename.yaml
```
# Configuration
## ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev
data:
  DB_HOST: mysql
  DB_PORT: "3306"
```
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod-config
 namespace: dev
spec:
  containers:
  - name: nginx-container
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
````
```bash
kubectl apply -f filename.yaml
kubectl get configmap -n dev
kubectl exec -it myapp-pod-config -n dev -- printenv
```
