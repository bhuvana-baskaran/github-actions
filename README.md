# github-actions
**.github/workflows/python-flask-app-sp.yaml**
this file works only with below configuration
- AKS cluster that has Local accounts with K8s RBAC enabled
- Azure Kubernetes Service Cluster Admin Role assigned to AKS for SP (az aks get-credentials --admin)
- Azure Kubernetes Service Cluster User Role to AKS for SP (az aks get-credentials)
- kubelogin is not required as the cluster doesn't have AAD authentication enabled
- In the Azure login workflow step, passed SP client id & client secret as json secret

For AKS to access ACR, either create imagePullSecret with

kubectl create secret docker-registry acr-secret \
  --docker-server=acr-name.azurecr.io \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email

or assign AcrPull role on ACR for UMI of aks nodepools (by default control plane has system assinged and nodepools have user assigned identity)

Here I have used service principle for azure login and deploy to AKS cluster

For AKS cluster with 'Microsoft Entra ID authentication with Azure RBAC' mode, assign below roles for SP to AKS
- Azure Kubernetes Service Cluster User Role
- Azure Kubernetes Service RBAC Writer / Admin
- add kubelogin action to install kubelogin which is required for AAD authentication
- disable admin login on AKS deploy stage
- in azure login 
  
**.github/workflows/python-flask-app-build.yaml** this file builds and push the image to ACR

**.github/workflows/python-flask-app-deploy-oidc.yaml** this file gets triggered when build completed and deploy to aks

I tried to use upload artifact in build and download artifact in deploy to get the build number. but it doesn't work if they are on the different workflows. it works only in the same workflow across different jobs.

So I got build number of build workflow using github.event.workflow_run.run_number default variable in deploy workflow. if you trigger deploy directly it will not return any output.

Instead of service principle I used OIDC authentication with managed identity here.

I used aks nodepool default UMI. added federated credentials for this UMI with github OIDC issuer URL, repo name, github user/org name & branch.

In my AKS cluster
- Enabled Microsoft AAD authentication with Azure RBAC enabled
- kubelogin is used in the workflow as the cluster have AAD authentication enabled
- In the Azure login workflow step, just used UMI client ID not secret
I gave below access to the UMI
- Azure Kubernetes Service Cluster User Role to AKS (this is for az aks get-credentials to work)
- Azure Kubernetes Service RBAC Writer (this is for deployment to succeed)

