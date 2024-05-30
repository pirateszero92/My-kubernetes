# Deployment Wordpress and Mysql on Kubernetes.

# 1. Create Namespace
	kubectl create ns wordpress

# 2. Create secret.yaml
	nano secret.yaml

#Add to file

	apiVersion: v1
	kind: Secret
	metadata:
 	  name: mysql-secret
  	  namespace: wordpress
	type: kubernetes.io/basic-auth
	stringData:
  	  password: MyP@sswo0rd!

#Apply secret

	kubectl apply -f secret.yaml


