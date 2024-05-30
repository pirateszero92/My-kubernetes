mkdir nfs-storage && cd nfs-storage
nano nsf-provisioner.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: nfs.csi.k8s.io
parameters:
  server: 192.168.210.252
  share: /volume1/k8s-data
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
  
kubectl apply -f nfs-provisioner.yaml
