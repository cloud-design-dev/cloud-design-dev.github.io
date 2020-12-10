---
layout: default
title: Deploy a Wireguard VPN Server in IBM Cloud VPC using Terraform 
parent: Terraform Examples 
nav_order: 2
---

# Wireguard Server on IBM Cloud VPC
The repository will spin up a new VPC on IBM Cloud and configure a Wireguard VPN server as well as additional compute nodes spread across the VPC region. This will allow us to connect to each of the compute instances via our Wireguard server as well as hit the VPC Cloud Service endpoints. 

## Step 1: Install Wireguard Tools
If you are on macOS you can use `brew` to install the Wireguard tools used to generate our client keys. 

```shell
brew install wireguard-tools
```

For most linux distributions you can install via the OS package manager:

```shell
[yum/apt-get] install wireguard-tools
```

## Step 2: Generate Wireguard Client Keys and Preshared Key
```
$ wg genkey | tee privatekey | wg pubkey | tee publickey
$ wg genpsk | tee presharedkey
```

## Step 3: Clone/Fork the repository

```
git clone https://github.com/cloud-design-dev/ibmcloud-vpc-wireguard.git
cd ibmcloud-vpc-wireguard
```

## Step 4: Update Credentials File 
The `credentials.tfvars` will hold all of the sensetive variables that get passed to our installer script:

**Copy example file** 
```shell
cp credentials-example credentials.tfvars
```

The `.gitignore` file has been configured to ignore any `.tfvars` files to prevent you from accidently pushing your Wireguard secrets to this repository. 

**Update `credentials.tfvars` file**

```shell
remote_ssh_ip           = "Your Local IP"
client_private_key      = "Client Private Key generated in Step 2"
client_public_key       = "Client Public Key generated in Step 2"
client_preshared_key    = "Client Preshared Key generated in Step 2"
resource_group          = "Resource Group where you will deploy VPC and resources"
region                  = "The IBM Cloud region where you will deploy the VPC and resources"
ssh_key                 = "SSH key to add to compute instances"
```

## Step 5: Initialize and Validate Terraform
```shell
$ terraform init
$ terraform validate 
```

If validation passes you can now proceed to generating the Terraform plan

## Step 6: Generate Terraform Plan

```shell
$ terraform plan -var-file="./credentials.tfvars" -out "default.tfplan"
```

If the plan generates successfully you can now run `apply` to deploy the resources. 

## Step 7: Deploy resources

```shell
$ terraform apply "default.tfplan"
```

After a successful deployment Terraform will generate the local Wireguard configuration needed to connect to our Wireguard instance. The file will saved as `${vpc_name}-wireguard.conf`. You will need to update it with the servers Public Key and Preshared Key. Once that is complete you can launch the macOS Wireguard app and import the tunnel. Click Activate to connect to your Wireguard VPC VPN server:

![Active Wireguard Tunnel](https://dsc.cloud/quickshare/wg-vpc-tunnel.png)