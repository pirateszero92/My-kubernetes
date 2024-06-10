# Install and setup nfs server

####Setup an NFS server#######

Caution: This section will show you how to configure a simple NFS server on Ubuntu for the purpose of this tutorial. This is not a production-grade NFS setup.

If you don’t have a suitable NFS server already, you can simply create one on a local machine with the following commands on Ubuntu:

	sudo apt-get install nfs-kernel-server

Create a directory to be used for NFS:

	sudo mkdir -p /share/nfs
	sudo chown nobody:nogroup /share/nfs
	sudo chmod 0777 /share/nfs

####sudo chown -R 999:999 /share/nfs##### work for mysql, postgresql 
 
####sudo chown nobody:nogroup /share/nfs#### not work for mysql, mariada,datbase

Edit the /etc/exports file. Make sure that the IP addresses of all your MicroK8s nodes are able to mount this share. For example, to allow all IP addresses in the 192.168.210.0/24 subnet:

	sudo mv /etc/exports /etc/exports.bak
	sudo echo '/share/nfs 192.168.210.0/24(rw,sync,no_subtree_check,insecure,no_root_squash)' | sudo tee /etc/exports
 
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
