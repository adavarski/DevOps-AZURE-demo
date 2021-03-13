### Demos:

[Create Linux(Ubuntu) VM](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/VM/Linux-VM)

[Create WindowsServer(2019-Datacenter) VM](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/VM/Windows-VM)

[Create Load Balancer + Linux VMs](https://github.com/adavarski/DevOps-AZURE-demo/tree/main/VM/LB)


### Quick terraform howto
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
