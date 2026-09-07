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
    resources:
      requests: {cpu: "250m", memory: "64Mi"}
      limits:   {cpu: "500m", memory: "128Mi"}
    livenessProbe:
      httpGet: {path: /, port: 80}
      initialDelaySeconds: 10
```
Then run kubectl apply command against this file.

```bash
kubectl apply -f pod.yaml
```