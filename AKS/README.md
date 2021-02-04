### Example AKS cluster using Terraform

This repository showcases using Terraform to provision a new AKS cluster with nodes within.

### Prerequisites:
Azure subscription: If you don't have an Azure subscription, create a free account before you begin.

- Install and configure az, terraform and kubectl (ensure that az, terraform and kubectl are installed first).

- Authenticate to Azure

Initialise the Azure CLI if you haven't already:
```
az login
```
- Create an Azure service principal using the Azure CLI
```
# Get subscriptionId
az account list --query "[].{name:name, subscriptionId:id}"
# Use subscriptionId for Azure service principal creation
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/{subscriptionId}"
```

- Authenticate to Azure using a service principal (optional)
```
az login --service-principal -u <service_principal_name> -p "<service_principal_password>" --tenant "<service_principal_tenant>"
```
- Set the current Azure subscription - for use if you have multiple subscriptions (optional)
```
az account list --query "[].{name:name, subscriptionId:id}"
az account set --subscription="<subscription_id>"
az account show
```
Generate public/private ssh key pair (needed to ssh login @ k8s pool VMs)
```
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/davar/.ssh/id_rsa): /home/davar/.ssh/azure-aks
...
```
Setup variables (Fill out variables.tf with the variables you'd like)

Required variables are location, and name, the Azure location/region, name you'd like your cluster, client_id, client_secret, subscription_id, tenant_id (Note: you get {client_id, client_secret, subscription_id, tenant_id} during service principal creation)

NB: Note Your free account has a four-core limit that will be violated if we go with the big AKS cluster, so if we want 2 nodes pool use: D1_v2, if 1 node pool:Standard_D2_v2 is OK.

Check available location and k8s versions for needed location (needed for variables.tf):
```
$ az account list-locations -o table
$ az aks get-versions --location eastus
```

- Provisioning
terraform init
terraform apply

- Configure kubectl
```
cp ~/.kube/config ~/.kube/config.BACKUP
az aks get-credentials --resource-group k8s-resources --name kubernetes-aks1
cat ~/.kube/config
```
- Test it works
```
kubectl get nodes -o wide
```
- Tearing down

```
terraform destroy
```

What now?
AKS documentation for more information on AKS itself.
Terraform AKS documentation for more details on customising the cluster.

