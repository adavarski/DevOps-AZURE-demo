### Example Azure Ubuntu virtual machine

#### Architecture diagram

Pre:
```
cd ~/Downloads
wget https://www.umlet.com/download/umlet_14_3/umlet-standalone-14.3.0.zip
unzip umlet-standalone-14.3.0.zip
sudo apt install pngquant
```
Generate aprchitecture.png:
```
make terraform-plan
```

![](architecture.png)

#### Usage (on a Ubuntu Desktop)

Install the tools:

```bash
./provision-tools.sh
```
Genrate ssh keys:
```
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/davar/.ssh/id_rsa): /home/davar/.ssh/azure-devops
```

Login into azure-cli:

```bash
az login
```

List the subscriptions and select the current one if the default is not OK:

```bash
az account list --all
az account set --subscription=<id>
az account show
```

Review `main.tf` and maybe change the `location` variable.

Initialize terraform:

```bash
make terraform-init
```
Terraform plan example:
```
export TF_VAR_admin_username="$USER"
export TF_VAR_admin_ssh_key_data="$(cat ~/.ssh/azure-devops.pub)"
terraform plan
```

Launch the example:

```bash
make terraform-apply
```

At VM initialization time [cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) will run the `provision-app.sh` script to launch the example application.

After VM initialization is done (check the boot diagnostics serial log for cloud-init entries), test the `app` endpoint:

```bash
wget -qO- "http://$(terraform output app_ip_address)/test"
```

And open a shell inside the VM:

```bash
ssh "$(terraform output app_ip_address)"
```
