#Get this Helm chart

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update

#Install Prometheus Helm Chart on Kubernetes Cluster  

    helm install kube-prom-stack prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

#Get the password for the admin user of the Grafana dashboard

    kubectl get secret --namespace monitoring kube-prom-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

#Create : prom-ingress.yaml

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: prom-ingress
      namespace: monitoring
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - host: npm.tvdirect.tv
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kube-prom-stack-grafana
                port:
                  number: 80
