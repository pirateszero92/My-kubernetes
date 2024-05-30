# Deploy dashboard:

  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

#Getting Access Dashboard

  kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard

#Change the Line

type: ClusterIP ---> type: LoadBalancer  wq!(save and exit)

#check status
  kubectl get all -n kubernetes-dashboard

##Create token:
  kubectl create serviceaccount admin -n kubernetes-dashboard
  kubectl create clusterrolebinding admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin
  kubectl -n kubernetes-dashboard create token admin

##Getting a long-lived Bearer Token for ServiceAccount

Create file : user-admin.yaml

  apiVersion: v1
  kind: Secret
  metadata:
    name: admin
    namespace: kubernetes-dashboard
    annotations:
      kubernetes.io/service-account.name: "admin"   
  type: kubernetes.io/service-account-token

kubectl apply -f user-admin.yaml

##and run 
  kubectl get secret admin -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d

Example:

Wqkn4pKv7KEh__2mQFz-DcjxHbdJqDiHsDuNu2MGJ85rimKSacr1eXwoNZT1uQBjz6wkIt5tFehQWlD6rdyItYVQ


###Clean up and next steps###

Remove the admin ServiceAccount and ClusterRoleBinding.

  kubectl -n kubernetes-dashboard delete serviceaccount admin
  kubectl -n kubernetes-dashboard delete clusterrolebinding admin
