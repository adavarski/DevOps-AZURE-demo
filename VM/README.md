Create Linux(Ubuntu) VM

Create Windows VM 

Create VMs+LB

----------
Install the tools:

../build-environment.sh

Login into azure-cli:
```
az login
```
List the subscriptions and select the current one if the default is not OK:
```
az account list --all
az account set --subscription=<id>
az account show
```
Review main.tf and maybe change the location variable.

Initialize terraform:
```
terraform init
```
Launch the example:
```
terraform apply
```

Clean environment:
```
terraform destroy
```
