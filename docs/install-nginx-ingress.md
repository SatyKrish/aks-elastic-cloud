# Create a namespace for your ingress resources

```
$ kubectl create namespace ingress
```

# Add the ingress-nginx repository

```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

# Use Helm to deploy an NGINX ingress controller

```
$ helm upgrade --install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```