---
layout: default
title: Virtual Private Cloud API
parent: IBM Cloud Rest API
nav_order: 2
---

# Overview
This page shows examples of interacting with the IBM Cloud [VPC API](https://cloud.ibm.com/apidocs/vpc).  

## Prerequisites
 - You have a valid IAM Token as outlined [here](./index.md)

### Endpoints
The endpoint is based on the region of the service and follows the convention https://REGION.iaas.cloud.ibm.com. For example, when IBM Cloud VPC is hosted in Dallas, the base URL is https://us-south.iaas.cloud.ibm.com. [VPC Region Endpoints](https://cloud.ibm.com/apidocs/vpc#endpoint-url)

### Versioning
API requests require a major version in the path (/v1/) and a date-based version as a query parameter in the format version=`YYYY-MM-DD`. You can use any date-based version up to the current date. Start development of new applications with the current date as a fixed value. See the [VPC API Change Log](https://cloud.ibm.com/docs/vpc?topic=vpc-api-change-log) for API changes. 


## Create a VPC

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