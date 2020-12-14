# AKS Elastic Cloud
A reference implementation for high available Elastic Cloud instance on Azure Kubernetes Service

## Deploy Elastic Operator

Install custom resource definitions and the operator with its RBAC rules:

```sh
$ kubectl apply -f https://download.elastic.co/downloads/eck/1.3.0/all-in-one.yaml
```
 
Monitor the operator logs:

```sh
$ kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
```

## Deploy Elasticsearch Cluster

Apply a simple Elasticsearch cluster specification, with one Elasticsearch node:

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.10.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
  http:
    tls:
      selfSignedCertificate:
        disabled: true
EOF
```

### Monitor cluster health and creation progressedit


### Request Elasticsearch accessedit

## Deploy Kibana Dashboard

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.10.0
  count: 1
  elasticsearchRef:
    name: quickstart
  http:
    tls:
      selfSignedCertificate:
        disabled: true
EOF
```

### Install Cert Issuer

```yaml
$ cat <<EOF | kubectl apply --validate=false -f -
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: demo@example.com
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: nginx
          podTemplate:
            spec:
              nodeSelector:
                "kubernetes.io/os": linux
EOF
```

### Expose Kibana Dashboard

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kibana-ingress
  annotations:
    kubernetes.io/ingress.class: nginx    
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - demo.siem-fortidm.com
    secretName: siem-tls-secret 
  rules:
  - host: demo.siem-fortidm.com
    http:
      paths:
      - path: /(.*)
        backend:
          serviceName: quickstart-kb-http
          servicePort: 5601
EOF
```