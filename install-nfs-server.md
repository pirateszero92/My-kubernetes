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
