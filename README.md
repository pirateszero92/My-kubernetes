# kubernetes Begining to Devops

Install Kubernetes Cluster on Unbuntu Server Step by Step.

1. Prepare 3 Server and Network IP.
    - K8s-Master-node 
    - K8s-Worker-node
    - NFS-server for Provisioner Persisten Volume
    3 IP for server and 1-150 for External IP (Loadbalancer)
        Example:
        192.168.210.250 k8s-master
        192.168.210.251 k8s-n1
        192.168.210.252 k8s-nfs
        IP range : 192.168.210.1-192.168.210.150

2. 
