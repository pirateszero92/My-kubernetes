# install-metallb

MetalLB can be deployed either with a simple Kubernetes manifest or with Helm. The rest of this example assumes MetalLB was deployed following the Installation instructions, and that the Ingress-Nginx Controller was installed using the steps described in the quickstart section of the installation guide.

MetalLB requires a pool of IP addresses in order to be able to take ownership of the ingress-nginx Service. This pool can be defined through IPAddressPool objects in the same namespace as the MetalLB controller. This pool of IPs must be dedicated to MetalLB's use, you can't reuse the Kubernetes node IPs or IPs handed out by a DHCP server.

Installation By Manifest

To install MetalLB, apply the manifest:

	kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml


Create metallb-loadbalancer.yaml

	apiVersion: metallb.io/v1beta1
	kind: IPAddressPool
	metadata:
	  name: default
	  namespace: metallb-system
	spec:
	  addresses:
	  - 192.168.210.1-192.168.210.150
	  autoAssign: true
	---
	apiVersion: metallb.io/v1beta1
	kind: L2Advertisement
	metadata:
	  name: default
	  namespace: metallb-system
	spec:
	  ipAddressPools:
	  - default
   
Apply config

	kubectl apply -f  metallb-loadbalancer.yaml

Check service

	kubectl -n ingress-nginx get svc

Example Output :

 	NAME                   TYPE          CLUSTER-IP     EXTERNAL-IP    PORT(S)
	default-http-backend   ClusterIP     10.0.64.249    <none>         80/TCP
	ingress-nginx          LoadBalancer  10.0.220.217   192.168.210.1  80:30100/TCP,443:30101/TCP

ref: https://metallb.universe.tf/installation/

ref: https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters

Page : Bare-metal considerations
