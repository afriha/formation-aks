# Namspace
To deploy a namespace, you need to create a YAML file named **namespace.yaml**. Use the example below:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
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
kubectl apply -f pod.yaml
```
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
# Manual Scheduling
## Node selector
```yaml
apiVersion:
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
## Taint and tolerations
Add a taint to your node
```bash
kubectl taint nodes node1 app=blue:NoSchedule
```
Apply a toleration to your pod
```yaml
apiVersion:
kind: Pod
metadata:
 name: myapp-pod
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