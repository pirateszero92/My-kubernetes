# Bare metal clusters
## This section is applicable to Kubernetes clusters deployed on bare metal servers, as well as "raw" VMs where Kubernetes was installed manually, using generic Linux distros (like CentOS, Ubuntu...)

	kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml

#Checking ingress controller version
Run /nginx-ingress-controller --version within the pod, for instance with kubectl exec:


# ref: https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters
