---
layout: default
title: Virtual Private Cloud API
parent: IBM Cloud Rest API
nav_order: 2
---

# Overview
This page shows examples of interacting with the IBM Cloud [VPC API](https://cloud.ibm.com/apidocs/vpc).  

**Table of Contents:**
- [Overview](#overview)
  * [Prerequisites](#prerequisites)
    + [Endpoints](#endpoints)
    + [Versioning](#versioning)
  * [Create a VPC](#create-a-vpc)
  * [Create a Public Gateway](#create-a-public-gateway)
  * [Create a Subnet Attached to a Public Gateway (Address Count)](#create-a-subnet-attached-to-a-public-gateway--address-count-)
  * [Create a Subnet Attached to a Public Gateway (CIDR)](#create-a-subnet-attached-to-a-public-gateway--cidr-)
  * [Create a Security Group with Rules](#create-a-security-group-with-rules)
  * [Create a Block Volume](#create-a-block-volume)
  * [Attach a Block Volume to a Compute Instance](#attach-a-block-volume-to-a-compute-instance)


## Prerequisites
 - You have a valid IAM Token as outlined [here](./index.md)

### Endpoints
The endpoint is based on the region of the service and follows the convention `https://REGION.iaas.cloud.ibm.com`. The examples on this page use `us-south` by default so my base API endpoint is `https://us-south.iaas.cloud.ibm.com`.  
 - [VPC Region Endpoints](https://cloud.ibm.com/apidocs/vpc#endpoint-url)

### Versioning
API requests require a major version in the path (/v1/) and a date-based version as a query parameter in the format version=`YYYY-MM-DD`. You can use any date-based version up to the current date. Start development of new applications with the current date as a fixed value. See the [VPC API Change Log](https://cloud.ibm.com/docs/vpc?topic=vpc-api-change-log) for API changes. 

## Create a VPC
VPC is a virtual network in IBM Cloud. It gives you cloud security, with the ability to scale dynamically, by providing fine-grained control over your virtual infrastructure and your network traffic segmentation.

 - **RESOURCE_GROUP_ID:** The ID of the resource group the VPC will be deployed in to. 

```shell
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/vpcs?version=YYYY-MM-DD&generation=2" \
-d '{
    "address_prefix_management": "auto",
    "name": "rest-demo-vpc",
    "resource_group": {
        "id": "RESOURCE_GROUP_ID"
    }
}' 
```

## Create a Public Gateway
This example will create a VPC [Public Gateway](https://cloud.ibm.com/docs/vpc?topic=vpc-about-networking-for-vpc#public-gateway-for-external-connectivity). A Public Gateway enables a subnet (or subnets) and all the attached virtual server instances to connect to the internet. Public gateways use Many-to-1 NAT.

 - **VPC_ID:** The ID of the VPC where the subnet will be created. 
 - **RESOURCE_GROUP_ID:** The ID of the resource group the VPC will be deployed in to. 

```shell
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/public_gateways?version=YYYY-MM-DD&generation=2" \
-d '{
	"name": "rest-demo-public-gateway",
	"vpc": {
		"id": "VPC_ID"
	},
	"resource_group": {
		"id": "RESOURCE_GROUP_ID"
	},
	"zone": {
		"name": "us-south-1"
	}
}'
```

## Create a Subnet Attached to a Public Gateway (Address Count)
This example uses `total_ipv4_address_count` to carve out a set number of IPs from the default [Address Prefix](https://cloud.ibm.com/docs/vpc?topic=vpc-vpc-behind-the-curtain#address-prefixes). If you need to provision a specific CIDR block use the example under this one. 

 - **NETWORK_ACL_ID:** The network ACL ID to use for this subnet. If unspecified, the default network ACL for the VPC is used.  
 - **VPC_ID:** The ID of the VPC where the subnet will be created.    
 - **PUBLIC_GATEWAY_ID:** The Public Gateway ID to use for this subnet.

```shell 
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/subnets?version=YYYY-MM-DD&generation=2" \
-d '{
    "name": "rest-demo-subnet-1",
    "total_ipv4_address_count": 32,
    "ip_version": "ipv4",
    "zone": {
        "name": "us-south-1"
    },
    "vpc": {
        "id": "VPC_ID"
    },
    "public_gateway": {
        "id": "PUBLIC_GATEWAY_ID"
    },
    "network_acl": {
        "id": "NETWORK_ACL_ID"
    }
}'
```

## Create a Subnet Attached to a Public Gateway (CIDR)
This example uses `ipv4_cidr_block` to carve out a specific CIDR block from the default [Address Prefix](https://cloud.ibm.com/docs/vpc?topic=vpc-vpc-behind-the-curtain#address-prefixes). 

 - **NETWORK_ACL_ID:** The network ACL ID to use for this subnet. If unspecified, the default network ACL for the VPC is used.  
 - **VPC_ID:** The ID of the VPC where the subnet will be created.    
 - **PUBLIC_GATEWAY_ID:** The Public Gateway ID to use for this subnet.

```shell
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/subnets?version=YYYY-MM-DD&generation=2" \
-d '{
    "name": "rest-demo-subnet-2",
    "ipv4_cidr_block": "10.240.0.0/24",
    "ip_version": "ipv4",
    "zone": {
        "name": "us-south-1"
    },
    "vpc": {
        "id": "VPC_ID"
    },
    "public_gateway": {
        "id": "PUBLIC_GATEWAY_ID"
    },
    "network_acl": {
        "id": "NETWORK_ACL_ID"
    }
}'
```

## Create a Security Group with Rules
A [security group](https://cloud.ibm.com/docs/vpc?topic=vpc-security-in-your-vpc#sgs-security) acts as a Stateful virtual firewall with a collection of rules that specify whether to allow or deny traffic for an associated compute instance. In this example we are allowing inbound ICMP and SSH and allowing all for outbound traffic.

 - **VPC_ID:** The ID of the VPC where the security group will be created. 

```shell 
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/security_groups?version=YYYY-MM-DD&generation=2" \
-d '{
	"name": "my-security-group",
	"rules": [{
			"direction": "inbound",
			"ip_version": "ipv4",
			"protocol": "tcp",
			"port_min": 22,
			"port_max": 22,
			"remote": {
				"cidr_block": "0.0.0.0/0"
			}
		},
		{
			"direction": "outbound",
			"ip_version": "ipv4",
			"protocol": "all",
			"remote": {
				"cidr_block": "0.0.0.0/0"
			}
		},
		{
			"direction": "inbound",
			"ip_version": "ipv4",
			"protocol": "icmp",
			"type": 8,
			"code": 0,
			"remote": {
				"cidr_block": "0.0.0.0/0"
			}
		}
	],
	"vpc": {
		"id": "VPC_ID"
	}
}'
```

## Create a Block Volume
Block Storage for VPC offers block-level volumes that are attached to an instance as a boot volume when the instance is created, or attached as secondary data volumes. 

 - **PROFILE_NAME:** The name of the volumes IOP profile. Default is `general-purpose`. See [IOP Tiers](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles#tiers) for the full list.
 - **RESOURCE_GROUP_ID:** The ID of the resource group the VPC will be deployed in to. 

```shell 
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/volumes?version=YYYY-MM-DD&generation=2" \
-d '{
  "name": "my-volume-1",
  "capacity": 500,
  "zone": {
    "name": "us-south-2"
  },
  "profile": {
    "name": "PROFILE_NAME"
  },
  "resource_group": {
    "id": "RESOURCE_GROUP_ID"
  }
}'
```

## Create a Compute Instance
This example will create a new VPC compute instance. 
 - **VPC_ID:** The ID of the VPC where the subnet will be created.
 - **SUBNET_ID:** The ID of the Subnet to use for the compute instance. 
 - **IMAGE_ID:** The ID of OS image to use.  
 - **SSH_KEY_ID:** The ID of the SSH key that will be added to the compute instance.

```shell
curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/instances?version=2020-12-01&generation=2" \
-d '{
    "boot_volume_attachment": {
      "volume": {
            "name": "rest-api-test-boot-volume",
            "profile": {
                "name": "general-purpose"
            }
        }
    },
    "primary_network_interface": {
      "name": "rest-api-test-nic",
      "subnet": {
        "id": "SUBNET_ID"
      }
    },
    "name": "rest-api-test-instance",
    "zone": {
      "name": "us-south-1"
    },
    "vpc": {
      "id": "VPC_ID"
    },
    "profile": {
      "name": "cx2-2x4"
    },
    "image": {
      "id": "IMAGE_ID"
    },
    "keys": [
      {
        "id": "SSH_KEY_ID"
      }
    ]
  }'
```

## Attach a Block Volume to a Compute Instance
This example will attach a block volume to a running compute instance. If you want the volume to remain after the instance is deleted change `"delete_volume_on_instance_delete": true` to `"delete_volume_on_instance_delete": false` when making the call.

```shell 
$ curl -X POST -H "Authorization: ${iam_token}" \
"https://us-south.iaas.cloud.ibm.com/v1/instances/INSTANCE_ID/volume_attachments?version=YYYY-MM-DD&generation=2" \
-d '{
    "delete_volume_on_instance_delete": true,
    "name": "my-volume-attachment-data-5iops",
    "volume": {
        "id": "VOLUME_ID"
    }
}'
```