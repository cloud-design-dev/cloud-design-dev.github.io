---
layout: default
title: Using the VPC CLI Plugin
parent: IBM Cloud CLI
nav_order: 2
---

# Overview
Examples of interacting with the IBM Cloud VPC CLI Plugin. 

## VPC

## Subnet

### Get all Subnets in a VPC and return their ID and Name

 - VPC_NAME: The name of the VPC where the subnets reside

```shell
ic is subnets --output json | jq -r '.[] | select(.vpc.name=="VPC_NAME") | .name,.id'
```


## Images

### Import custom image in to VPC
 - CUSTOM_IMAGE_NAME: The name assigned to the imported image  
 - IMAGE: The file name of the `qcow2` image  
 - OS_NAME: The IBM Cloud equivalent OS Name. See [here](#finding-ibm-image-os-names) for supported options.  
 - RESOURCE_GROUP_ID: The resource group ID for the imported image

```shell
ibmcloud is image-create CUSTOM_IMAGE_NAME --file cos://region/bucket/IMAGE --os-name OS_NAME --resource-group-id RESOURCE_GROUP_ID
```

## Instances 
Examples for interacting with VPC compute instances

### Get Windows instance password

 - INSTANCE_ID: The compute instance ID

```shell
ibmcloud is instance-initialization-values INSTANCE_ID --private-key @/path/to/private_key
```

### Get primary IP from instance 
 - INSTANCE_ID: The compute instance ID

```shell
ibmcloud is instance INSTANCE_ID --json | jq -r '.primary_network_interface.primary_ipv4_address'
```

### Grab ID of compute instance based on name
 - NAME_OF_INSTANCE: The name of the compute instance

```shell
ibmcloud is instances --output json | jq -r '.[] | select(.name=="NAME_OF_INSTANCE") | .id'
```

### Find all networking interfaces attached to instance and return their name and ID
 - INSTANCE_ID: The compute instance ID

```shell
ibmcloud is in-nics INSTANCE_ID --output json | jq -r '.[] | .name,.id'
```

### Find the floating IP attached to a specific compute instance
 - INSTANCE_ID: The compute instance ID

```shell
ibmcloud is instance INSTANCE_ID --output json | jq -r '.network_interfaces[].floating_ips[].id'
```

## Finding Names and IDs

#### Finding IBM Image OS Names
You can run the following command to list the supported OS Names:

```shell
ibmcloud is images --visibility public --json | jq -r '.[] | select(.status=="available") | .operating_system.name'
```
