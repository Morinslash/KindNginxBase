# Generic Setup for the Local Kind Cluster with Nginx

Run cluster
```bash
kind create cluster --config .kind-config/cluster-config.yml
```

Install Nginx Ingress in the Cluster
```bash
helm install ingress oci://ghcr.io/nginxinc/charts/nginx-ingress --version 1.2.1 --namespace ingress --create-namespace --set controller.service.type=NodePort --set controller.service.httpPort.nodePort=30950
```

Deploy sample app and add `app-nginx.local-com` to the `hosts` file
```bash
kubectl create namespace app-nginx
kubectl apply -f .\sample-app.yml -n app-nginx
```
```txt
127.0.0.1 app-nginx.local-com
```

Access sample nginx app on `http://app-nginx.local-com:8080`


## ArgoCD setup

Add Argo Helm repo
```bash
helm repo add argo https://argoproj.github.io/argo-helm

helm repo update
```
Install ArgoCD
```bash
helm install argocd argo/argo-cd -f .\argo-cd\argocd-values.yml --namespace argocd --create-namespace
```

Get ArgoCD password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Add new DNS to `hosts` file and access ArgoCD over HTTP
```txt
127.0.0.1 argocd.local-com 
```

`http://argocd.local-com`

## Example setup of the application
```yaml
project: default
destination:
  server: 'https://kubernetes.default.svc'
  namespace: sample-nginx
syncPolicy:
  syncOptions:
    - CreateNamespace=true
sources:
  - repoURL: 'https://github.com/Morinslash/KindNginxBase'
    path: helm-charts/sample-nginx
    targetRevision: HEAD
    helm:
      valueFiles:
        - $values/remote-values.yml
  - repoURL: 'https://github.com/Morinslash/sample-nginx-config-kind'
    ref: values
```
