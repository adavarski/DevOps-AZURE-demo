#### Install azure-cli, terraform, kubectl, needed packages

```
$ ./build-environment.sh
```
#### Simple examples:

- [Azure AKS (managed k8s)](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/AKS)

- [Linux(Ubuntu) VM](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/VM/Linux-VM)

- [WindowsServer(2019-Datacenter) VM](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/VM/Windows-VM)

- [Load Balancer + Linux VMs](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/VM/LB)


### Configuring Terraform for Azure
Before writing Terraform code to provision a cloud infrastructure such as Azure, we must configure Terraform to allow the manipulation of resources in an Azure subscription. To do this, we will first create a new Azure service principal (SP) in Azure Active Directory (AD), which, in Azure, is an application user who has permission to manage Azure resources.

For more details about the Azure SP, read the documentation here: https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals .
For this Azure SP, we have to assign to it the contributing permissions on the subscription in which we will create resources.

#### Creating the Azure SP
This operation can be done either via the Azure portal (all steps are detailed on the official documentation here: https://docs.microsoft.com/en-us/azure/active- directory/develop/howto-create-service-principal-portal ) or via a script by executing the following az cli command (which we can launch in Azure Cloud Shell).
The following is a template az cli script that you have to run for creating an service principal, where you have to enter your SP name, role, and scope:
```
az ad sp create-for-rbac --name="<ServicePrincipal name>" --role="Contributor" --scopes="/subscriptions/<subscription Id>"
```
See the following example:
```
az ad sp create-for-rbac --name="SPForTerraform" --role="Contributor" --scopes="/subscriptions/8921-1444-..."
```
This sample script creates a new service principal named SPForTerraform and gives it the contributor permission on the subscription ID, 8921... .

For more details about the Azure CLI command to create an Azure SP, see
the documentation: https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest .

The creation of this service principal returns three pieces of identification information:
- The application ID, also called the client ID
- The client secret
- The tenant ID

Here, we have just seen how to create a service principal in the Azure AD and we have given it the permission to manipulate the resources of our Azure subscriptions. Now, let's see how to configure Terraform to use our Azure SP.


#### Configuring the Terraform provider
Once the Azure SP has been created, we will configure our Terraform code to connect to Azure with this SP. For this, follow these steps:
1. In a directory of your choice, create a new filename, provider.tf ( extension.tf corresponds to Terraform files), which contains the following
code:
```
provider "azurerm" {
subscription_id = "<subscription ID>"
client_id = "<Client ID>"
client_secret = "<Client Secret>"
tenant_id = "<Tenant Id>"
}
```
In this code, we indicate that the provider we are using is azurerm and that the authentication information to Azure is the service principal created. However, for security reasons, it is not advisable to put identification information in code, knowing that this code may be accessible by other people.
2. We will, therefore, improve the previous code by replacing it with this one:
```
provider "azurerm" {}
```
3. So, we delete the credentials in the Terraform code and we will pass the identification values to specific Terraform environment variables:
```
ARM_SUBSCRIPTION_ID
ARM_CLIENT_ID
ARM_CLIENT_SECRET
ARM_TENANT_ID
```

As a result, the Terraform code no longer contains any identification information. We have just seen how to configure Terraform for Azure authentication. We will now explain how to quickly configure Terraform to perform local development and testing.

#### Terraform configuration for local development and testing

When you work locally and want to test the Terraform code quickly, in a sandbox environment, for example, it may be more convenient and faster to use your own Azure account instead of using an SP. To do this, it is possible to connect to Azure beforehand with the az login command, and then enter your identification information in the window that opens.

If several subscriptions are accessed, the desired one can be selected with the following command:
```
az account set --subscription="<Subscription ID>"
```
Then, we configure the Terraform provider as before with the provider, "azurerm" { } . Of course, this authentication method is not to be done in the case of execution on a remote server. For more information on the provider configuration, refer to the documentation: https://www.terraform.io/docs/providers/azurerm/
index.html .

The Terraform configuration for Azure is, therefore, defined by the configuration of the provider that uses the information from an Azure SP. Once this configuration is complete, we can start writing Terraform code to manage and provision Azure resources.

#### Deploying the infrastructure with Terraform

With the Terraform code written, we now need to run Terraform to deploy our
infrastructure.
However, before any execution, it is necessary to first provide authentication with the
Azure SP to ensure that Terraform can manage Azure resources.
For this, we can either set the environment variables specific to Terraform to contain the
information of the SP created earlier in the Configuring Terraform for Azure section or use
the az cli script.
The following script exports the four Terraform environment variables in the Linux OS:
```
export ARM_SUBSCRIPTION_ID=xxxxx-xxxxx-xxxx-xxxx
export ARM_CLIENT_ID=xxxxx-xxxxx-xxxx-xxxx
export ARM_CLIENT_SECRET=xxxxxxxxxxxxxxxxxx 
export ARM_TENANT_ID=xxxxx-xxxxx-xxxx-xxxx
```
We can also use the az cli script with the login command:
```
az login
```
Once authenticated, we can run Terraform.
```
terraform fmt
terraform init
terraform validate
terraform plan -out=out.tfplan
terraform apply --auto-approve out.tfplan
```

Note: Protecting tfstate in a remote backend

When Terraform handles resources, it writes the state of these resources in a tfstate file. This file is in JSON format and preserves the resources and their properties throughout the execution of Terraform. By default, this file, called terraform.tfstate , is created locally when the first execution
of the apply command is executed. It will then be used by Terraform each time the plan command is executed in order to compare its state (written in this tfstate) with that of the target infrastructure, and hence return the preview of what will be applied. When using Terraform in an enterprise, this locally stored tfstate file poses many problems:
- Knowing that this file contains the status of the infrastructure, it should not be
deleted. If deleted, Terraform may not behave as expected when it is executed.
- It must be accessible at the same time by all members of the team handling
resources on the same infrastructure.
- This file can contain sensitive data, so it must be secure.
- When provisioning multiple environments, it is necessary to be able to use
multiple tfstate files.
With all of these points, it is not possible to keep this tfstate file locally or even to archive it in an SCM. To solve this problem, Terraform allows this tfstate file to be stored in a shared and secure storage called the remote backend. Terraform supports several types of remote backends; the list is available
here: https://www.terraform.io/docs/backends/types/remote.html.

In our case, we can use an azurerm remote backend to store our tfstates files with a storage account and a blob for the tfstate file.
We will, therefore, implement and use a remote backend in three steps:
1. The creation of the storage account
2. Terraform configuration for the remote backend
3. The execution of Terraform with the use of this remote backend
```
Let's look in detail at the execution of these steps:
1. To create an Azure Storage Account and a blob container, we can use either the
Azure portal ( https://docs.microsoft.com/en-gb/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal ) or an az cli script:
# 1-Create resource group
az group create --name MyRgRemoteBackend --location westeurope
# 2-Create storage account
az storage account create --resource-group MyRgRemoteBackend --name storageremotetf --sku Standard_LRS --encryption-services blob
# 3-Get storage account key
ACCOUNT_KEY=$(az storage account keys list --resource-group MyRgRemoteBackend --account-name storageremotetf --query [0].value -o tsv)
# 4-Create blob container
az storage container create --name tfbackends --account-name storageremotetf --account-key $ACCOUNT_KEY
This script creates a MyRgRemoteBackend resource group and a storage account, storageremotetf . Then, the script retrieves the key account from the storage account and creates a blob container, tfbackends , in this storage account. This script can be run in Azure Cloud Shell, and the advantage of using a script rather than using the Azure portal is that this script can be integrated into a
CI/CD process.
2. Then, to configure Terraform to use the previously created remote backend, we must add the configuration section in the Terraform.tf file:
terraform {
   backend "azurerm" {
      storage_account_name = "storageremotetfdemo"
      container_name = "tfbackends"
      key = "myappli.tfstate"
 }
}


The storage_account_name property contains the name of the storage account, the container_name property contains the container name, and the key property contains the name of the blob tfstate object.

However, there is still one more configuration information to be provided to Terraform so that it can connect and have permissions on the storage account.
This information is the access key, which is a private authentication and authorization key on the storage account. To provide the storage key to
Terraform, as with the Azure SP information, set an ARM_STORAGE_KEY environment variable with its value.

Note: Terraform supports other types of authentication on the storage account such as the use of a SAS token or by using an service principal. For more
information on configuring Terraform for an azurerm remote backend, refer to the documentation: https://www.terraform.io/docs/backends/types/azurerm.html.

3. Finally, once the Terraform configuration is completed, Terraform can be run with this new remote backend. It is during init that Terraform initializes the
context of the tfstate file and, by default, the init command remains unchanged with terraform init .

However, if several tfstates in several environments are used, it is possible to create several
remote backend configurations with the simplified code in the .tf file:

terraform {
backend "azurerm" {}
}

Then, create several backend.tfvars files that only contain the properties of the
backends.
These backend properties are the storage account name, the name of the blob container,
and the blob name of the tfstate:
storage_account_name = "storageremotetf"
container_name = "tfbackends"
key = "myappli.tfstate"

In this case, when executing the init command, we can specify the backend.tfvars file to use with the following command:

export ARM_ACCESS_KEY=xxxxxx
terraform init -backend-config="backend.tfvars"

In this execution, we can see the export of the ARM_ACCESS_KEY environment variable, as well as the Terraform init command that determines the backend configuration with the -backend-config option.

The -backend-config argument is the path to the backend configuration file. Personally, I prefer this way of doing things as it allows me to decouple the code by
externalizing the values of the backend properties and for better readability of the code.

With this remote backend, the tfstate file will no longer be stored locally, but on a storage account, which is a shared space. Therefore, it can be used at the same time by several users. This storage account offers, at the same time, security to protect the sensitive data of the tfstate but also the possibilities of backups and restoration of the tfstate files, which are an essential and critical element of Terraform as well.

```
