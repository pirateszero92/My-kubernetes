# kubernetes Begining to Devops

Install Kubernetes Cluster on Unbuntu Server Step by Step.

# 1. Prepare 3 Server and Network IP.
    - K8s-Master-node 
    - K8s-Worker-node
    - NFS-server for Provisioner Persisten Volume
        3 IP for server and 1-150 for External IP (Loadbalancer) 
    Example: 
        192.168.210.250 k8s-master 
        192.168.210.251 k8s-n1 
        192.168.210.252 k8s-nfs 
        IP range : 192.168.210.1-192.168.210.150

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

# Step 7: Initialize the Kubernetes cluster on the master node
    sudo kubeadm config images pull
    sudo kubeadm init --pod-network-cidr=10.1.0.0/16

    mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Step 8: Configure kubectl and Calico
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/tigera-operator.yaml
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/custom-resources.yaml -O
    sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.1.0.0\/16/g' custom-resources.yaml

    kubectl create -f custom-resources.yaml


# Step 9: Add worker nodes to the cluster
    sudo kubeadm join

# Step 10: Verify the cluster and test
    kubectl get node
    kubectl get po -A
