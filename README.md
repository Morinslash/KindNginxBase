# Generic Setup for the Local Kind Cluster with Nginx

Run cluster
```bash
kind create cluster --config .\cluster-config.yml
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
