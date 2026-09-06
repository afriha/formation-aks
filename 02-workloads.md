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