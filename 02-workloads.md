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
  name: index-html-configmap
  namespace: dev
data:
  index.html: |
    <h1> Hello formation </h2>
---
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod-config
 namespace: dev
spec:
  volumes:
  - name: nginx-index-file
    configMap:
      name: index-html-configmap
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: nginx-index-file # Must match the volume name above
      mountPath: /usr/share/nginx/html/
```
```bash
kubectl apply -f filename.yaml
kubectl get configmap -n dev
kubectl exec myapp-pod-config -n dev -- curl localhost:80
```
# Services
## NodePort
We use NodePort to expose our apps at the node level
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod-config
  namespace: dev
  labels:
    env: formation # Important for selectors
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    env: formation
    app: nginx
  name: myapp-pod-expose
  namespace: dev
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    env: formation # We have to match pod labels
    app: nginx
  type: NodePort
```
```bash
kubectl apply -f filename.yaml
kubectl get services -n dev
curl IP:NODEPORT # You can also access it on your browser

# On a deeper level, check your iptables
sudo nft list table ip nat

```
## Ingress

# Helm
Helm is a package manager for Kubernetes. Install it with this command:
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

We will now deploy the Ingress Controller:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx --create-namespace --namespace ingress-controller
```
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingressapp-demo
---
apiVersion: v1
kind: ConfigMap
metadata:
 name: index-html-main
 namespace: ingressapp-demo
data:
 index.html: |
   <html>
   <h1>Welcome to Ooredoo Kubernetes training</h1>
   </br>
   <h2>Hi! This is the main page of the app </h2>
   </html
---
apiVersion: v1
kind: ConfigMap
metadata:
 name: index-html-doc
 namespace: ingressapp-demo
data:
 index.html: |
   <html>
   <h1>Welcome to Ooredoo Kubernetes training</h1>
   </br>
   <h2>Hi! This is the doc page of the app </h2>
   </html
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: mainpage
  name: ingressdemoapp-main
  namespace: ingressapp-demo
spec:
  volumes:
  - name: nginx-index-file
    configMap:
      name: index-html-main  
  containers:
  - image: nginx
    name: ingressdemoapp-main
    volumeMounts:
    - name: nginx-index-file
      mountPath: /usr/share/nginx/html/
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: docpage
  name: ingressdemoapp-doc
  namespace: ingressapp-demo
spec:
  volumes:
  - name: nginx-index-file
    configMap:
      name: index-html-doc  
  containers:
  - image: nginx
    name: ingressdemoapp-doc
    volumeMounts:
    - name: nginx-index-file
      mountPath: /usr/share/nginx/html/
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
Then we will add services, pointing to those 2 pods:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: docpage
  name: docpage-svc
  namespace: ingressapp-demo
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: docpage
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: rootpage
  name: mainpage-svc
  namespace: ingressapp-demo
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: mainpage
```
Now we want to use an ingress so that the main page is available on the `/main` path and the doc page is available on the `/doc` path.
We will create the ingress manifest with 2 rules:

```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-demo
  namespace: ingressapp-demo
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /main(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: mainpage-svc
            port: 
              number: 80
      - path: /doc(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: docpage-svc
            port: 
              number: 80
      - path: /(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: mainpage-svc
            port:
              number: 80
```
