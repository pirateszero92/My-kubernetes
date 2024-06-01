# kubernetes Bare-metal on Virtualization.

## Install Kubernetes Cluster Docker Engine on Unbuntu Server Step by Step.

# 1. Prepare 3 Server and Network IP.
    - K8s-Master-node 
    - K8s-Worker-node
    - NFS-server for Provisioner Persisten Volume
    - docker server for nginx proxy manager and more
        4 IP for server and 1-150 for External IP (Loadbalancer) 
    Example: 
        192.168.210.250 k8s-master 
        192.168.210.251 k8s-n1 
        192.168.210.252 k8s-nfs
        IP range : 192.168.210.1-192.168.210.150
	192.168.210.200 docker

# 2. Install Kubernetes step by step

# Step 0: Static IP 
    sudo nano /ect/netplan/01-network-manager-all.yaml 

##Add to line

    #This is the network config written by 'subiquity'
    network: 
      ethernets: 
        eth0: 
          addresses: 
          - 192.168.210.250/24 
          nameservers: 
            addresses: 
            - 8.8.8.8 
            search: 
            - local 
          routes: 
          - to: default 
            via: 192.168.210.2 
      version: 2
      
# Step 1: Disable swap
    sudo swapoff -a
    sudo sed -i '/swap/s/^/#/' /etc/fstab
    sudo swapon --show

# Step 2: Setup hostnames
    sudo hostnamectl set-hostname "k8s-master"
    exec bash

# Step 3: Update the /etc/hosts File for Hostname Resolution
    127.0.1.1 k8s-master
    192.168.210.250 k8s-master
    192.168.210.251 k8s-n1
    192.168.210.252 k8s-nfs


# Step 4: Set up the IPV4 bridge on all nodes
#To configure the IPV4 bridge on all nodes, execute the following commands on each node.

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

#apply

    sudo modprobe overlay
    sudo modprobe br_netfilter

#sysctl params required by setup, params persist across reboots

    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

#Apply sysctl params without reboot
    sudo sysctl --system

# Step 5: Install kubelet, kubeadm, and kubectl on each node
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl
    mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt install kubeadm kubelet kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

# Step 6: Install Docker
    sudo apt install docker.io
    sudo mkdir /etc/containerd
    sudo sh -c "containerd config default > /etc/containerd/config.toml"
    sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
    sudo systemctl restart containerd.service
    sudo systemctl restart kubelet.service
    sudo systemctl enable kubelet.service

# Initialize the Kubernetes cluster on the master node

# Step 1: Initialize master node
    sudo kubeadm config images pull
    sudo kubeadm init --pod-network-cidr=10.1.0.0/16

    mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Step 2: Configure kubectl and Calico
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/tigera-operator.yaml
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/custom-resources.yaml -O
    sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.1.0.0\/16/g' custom-resources.yaml

    kubectl create -f custom-resources.yaml
    
# Step 3: For worker nodes to join the cluster
    sudo kubeadm join

# Step 4: Verify the cluster and test
    kubectl get node
    kubectl get po -A

# Next Step

# 1. install-helm

Install Helm from Script

Helm now has an installer script that will automatically grab the latest version of Helm and install it locally.

You can fetch that script, and then execute it locally. It's well documented so that you can read through it and understand what it is doing before you run it.

	curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

	chmod 700 get_helm.sh

	./get_helm.sh

ref: https://helm.sh/docs/intro/install/
   
# 2. install-ingress

#Bare metal clusters

This section is applicable to Kubernetes clusters deployed on bare metal servers, as well as "raw" VMs where Kubernetes was installed manually, using generic Linux distros (like CentOS, Ubuntu...)

	kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml

#Checking ingress controller version Run /nginx-ingress-controller --version within the pod, for instance with kubectl exec:


ref: https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters

# 3. install-metallb

MetalLB can be deployed either with a simple Kubernetes manifest or with Helm. The rest of this example assumes MetalLB was deployed following the Installation instructions, and that the Ingress-Nginx Controller was installed using the steps described in the quickstart section of the installation guide.

MetalLB requires a pool of IP addresses in order to be able to take ownership of the ingress-nginx Service. This pool can be defined through IPAddressPool objects in the same namespace as the MetalLB controller. This pool of IPs must be dedicated to MetalLB's use, you can't reuse the Kubernetes node IPs or IPs handed out by a DHCP server.

Installation By Manifest

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


# Install and setup nfs server

####Setup an NFS server#######

Caution: This section will show you how to configure a simple NFS server on Ubuntu for the purpose of this tutorial. This is not a production-grade NFS setup.

If you don’t have a suitable NFS server already, you can simply create one on a local machine with the following commands on Ubuntu:

	sudo apt-get install nfs-kernel-server

Create a directory to be used for NFS:

	sudo mkdir -p /share/nfs
	sudo chown nobody:nogroup /share/nfs
	sudo chown -R 999:999 /share/nfs
 
####sudo chmod 0777 /share/nfs#### not work for mysql, mariada,datbase

Edit the /etc/exports file. Make sure that the IP addresses of all your MicroK8s nodes are able to mount this share. For example, to allow all IP addresses in the 192.168.210.0/24 subnet:

	sudo mv /etc/exports /etc/exports.bak
	echo '/share/nfs 192.168.210.0/24(rw,sync,no_subtree_check)' | sudo tee /etc/exports
 
Finally, restart the NFS server:

	sudo systemctl restart nfs-kernel-server

####Install the CSI driver for NFS########

ref: https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/docs/install-csi-driver-v4.7.0.md


# Install NFS CSI driver v4.7.0 version on a kubernetes cluster

If you have already installed Helm, you can also use it to install this driver. Please check Installation with Helm.

Install with kubectl

Option 1. remote install

	curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/v4.7.0/deploy/install-driver.sh | bash -s v4.7.0 --


Option 2. local install

	git clone https://github.com/kubernetes-csi/csi-driver-nfs.git
 
	cd csi-driver-nfs
 
	./deploy/install-driver.sh v4.7.0 local


###check pods status:

	kubectl -n kube-system get pod -o wide -l app=csi-nfs-controller
	kubectl -n kube-system get pod -o wide -l app=csi-nfs-node
 
#example output:

	NAME                                       READY   STATUS    RESTARTS   AGE     IP             NODE
	csi-nfs-controller-56bfddd689-dh5tk       4/4     Running   0          35s     10.240.0.19    k8s-agentpool-22533604-0
	csi-nfs-node-cvgbs                        3/3     Running   0          35s     10.240.0.35    k8s-agentpool-22533604-1
	csi-nfs-node-dr4s4                        3/3     Running   0          35s     10.240.0.4     k8s-agentpool-22533604-0

####clean up NFS CSI driver#####

Option 1. remote uninstall

	curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/v4.7.0/deploy/uninstall-driver.sh | bash -s v4.7.0 --

Option 2. local uninstall

	git clone https://github.com/kubernetes-csi/csi-driver-nfs.git
 
	cd csi-driver-nfs

	git checkout v4.7.0
 
	./deploy/uninstall-driver.sh v4.7.0 local

ref: https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/docs/install-csi-driver-v4.7.0.md

# Provisioner NFS

Create folder nfs-storage && cd nfs-storage

    nano nsf-provisioner.yaml

Add Line:

	apiVersion: storage.k8s.io/v1
	kind: StorageClass
	metadata:
	  name: nfs-storage
	provisioner: nfs.csi.k8s.io
	parameters:
	  server: 192.168.210.252
	  share: /share/nfs
	reclaimPolicy: Delete
	volumeBindingMode: Immediate
	mountOptions:
	  - hard
	  - nfsvers=4.1

Apply:

	kubectl apply -f nfs-provisioner.yaml

Check Persistant Provisioner

	kubectl get pv
