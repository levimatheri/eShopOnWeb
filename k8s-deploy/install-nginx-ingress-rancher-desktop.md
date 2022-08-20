# How to install and configure nginx ingress on Rancher Desktop

## Disable Traefik
Uncheck `Enable Traefik` from the `Kubernetes Settings` page to disable Traefik. You may need to exit and restart Rancher Desktop for the change to take effect.

## Install NGINX ingress controller
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/cloud/deploy.yaml`

## Wait for ingress pods to come up
`kubectl get pods --namespace=ingress-nginx`

## Create a deployment [with service]
Example deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sampleapp
  labels:
    app: sampleapp
spec:
  replicas: 1
  template:
    metadata:
      name: sampleapp
      labels:
        app: sampleapp
    spec:
      containers:
      - name: sampleapp
        image: nginxdemos/hello
        ports:
          - containerPort: 80
  selector:
    matchLabels:
      app: sampleapp
---
apiVersion: v1
kind: Service
metadata:
  name: sampleapp
spec:
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: sampleapp
```    

## Create an ingress resource
Example ingress:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  labels:
    name: my-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: demo.localdev.me
    http:
      paths:
      - pathType: Prefix
        path: /myapp(/|$)(.*)
        backend:
          service:
            name: sampleapp
            port: 
              number: 80
      - pathType: Prefix
        path: /(/|$)(.*)
        backend:
          service:
            name: sampleapp
            port: 
              number: 80
```              

## Forward a local port to the ingress controller
`kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80`

> You should now be able to access your app through http://demo.localdev.me:8080/ or http://demo.localdev.me:8080/myapp