# github-actions
**.github/workflows/python-flask-app.yaml**
this file works only with below configuration
- AKS cluster that has Local accounts with K8s RBAC enabled
- Azure Kubernetes Service Cluster Admin Role assigned to AKS for SP (az aks get-credentials --admin)
- Azure Kubernetes Service Cluster User Role to AKS for SP (az aks get-credentials)
- kubelogin is not required as the cluster doesn't have AAD authentication enabled

For AKS to access ACR, either create imagePullSecret with
kubectl create secret docker-registry acr-secret \
  --docker-server=acr-name.azurecr.io \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email

or assign AcrPull role on ACR for UMI of aks nodepools (by default control plane has system assinged and nodepools have user assigned identity)

Here I have used service principle for azure login and deploy to AKS cluster
