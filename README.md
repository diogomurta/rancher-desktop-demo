# rancher-desktop-demo


## To get the Node IP
```bash
NODENAME=$(kubectl get nodes  -o json | jq -r '.items[].metadata.name')
```
```bash
IP=$(kubectl get node/$NODENAME -o json | jq -r '.status.addresses[] | select(.type=="InternalIP").address')
```
## To create a namespace and install a `rancher-hello-world` example
```bash
kubectl create ns demo-rancher
```

```bash
cat << EOF | kubectl apply -n demo-rancher -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: rancher-hello-world
  name: rancher-hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher-hello-world
  template:
    metadata:
      labels:
        app: rancher-hello-world
    spec:
      containers:
        - image: rancher/hello-world
          name: rancher-hello-world
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: rancher-hello-world-svc
spec:
  type: ClusterIP
  selector:
    app: rancher-hello-world
  ports:
    - port: 80
---    
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rancher-hello-world-http
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web
spec:
  rules:
    - host: rancher-hello-world.$IP.nip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: rancher-hello-world-svc
                port:
                  number: 80 
EOF
```
## echo it
```bash
echo rancher-hello-world.$IP.nip.io
```
## Curl it 
```bash
curl rancher-hello-world.$IP.nip.io
```


## To delete
```bash
kubectl delete all,ingress --all -n demo
```
