---
layout: default
title: Private DNS API
parent: IBM Cloud Rest API
nav_order: 3
---

# Overview
This page shows examples of interacting with the IBM Cloud [Private DNS](https://cloud.ibm.com/apidocs/dns-svcs#introduction-to-dns-services-api) API.  

**Table of Contents:**
- [Overview](#overview)
  * [Prerequisites](#prerequisites)
    + [Authentication](#authentication)
  * [Create a Private DNS Instance](#create-private-dns-instance)
  * [Create a Private DNS Zone](#create-a-private-dns-zone)

## Prerequisites

 - You have a valid IAM Token as outlined [here](./index.md)

### Authentication

Access to DNS Services instances for users in your account is controlled by IBM Cloud Identity and Access Management (IAM). Every user that accesses the DNS Services in your account must be assigned an access policy with an IAM role defined. Pass a bearer token in an authorization header. Tokens support authenticated requests without embedding service credentials in every call. For more information, see [Managing access with IAM for DNS Services](https://cloud.ibm.com/docs/dns-svcs?topic=dns-svcs-iam).

## Create Private DNS Instance

We will need to use the [Resource Controller](https://cloud.ibm.com/apidocs/resource-controller/resource-controller) API in order to provision an instance of the Private DNS offering.

 - **RESOURCE_GROUP_ID:** The ID of the resource group the VPC will be deployed in to. 
 - **RESOURCE_PLAN_ID:** The ID of the specific catatlog plan ID for the service. To find available options run `ibmcloud catalog service dns-svcs`.

```shell
curl -X POST "https://resource-controller.cloud.ibm.com/v2/resource_instances" \
-H "Authorization: ${iam_token}" -H "Content-Type: application/json"\
-d '{
    "name": "pdns-rest-test",
    "target": "bluemix-global",
    "resource_group": "RESOURCE_GROUP",
    "resource_plan_id": "RESOURCE_PLAN_ID",
    "tags": [
      "service:dns-svcs"
    ]
  }

```

## Create a Private DNS Zone

The following example will create a new [DNS Zone](https://cloud.ibm.com/docs/dns-svcs?topic=dns-svcs-dns-concepts#what-is-zone) in our Private DNS instance.

 - **DNS_INSTANCE:** The Private Instance ID. You can get this by running the command `ibmcloud dns instances`

```shell
curl -X POST -H "Content-Type: application/json" -H "Authorization: ${iam_token}" \
"https://api.dns-svcs.cloud.ibm.com/v1/instances/DNS_INSTANCE/dnszones" \
-d '{
	  "name": "pdns-rest.test",
	  "description": "Example Private DNS Zone",
	  "label": "us-east"
}'
```

## Add a Permitted Network to a DNS zone

 - **DNS_INSTANCE:** The Private Instance ID. You can get this by running the command `ibmcloud dns instances`.
 - **DNS_ZONE_ID:** The DNS Zone ID. 
 - **VPC_CRN:** The CRN of the VPC to add as a permitted network. 

```shell
curl -X POST -H "Content-Type: application/json" -H "Authorization: ${iam_token}" \
"https://api.dns-svcs.cloud.ibm.com/v1/instances/DNS_INSTANCE/dnszones/DNS_ZONE_ID/permitted_networks" \
-d '{
	  "permitted_network": {
		    "vpc_crn": "VPC_CRN"
		
	  }, 
	  "type": "vpc"
}'
```

## Links
 - [Private DNS API Docs](https://cloud.ibm.com/apidocs/dns-svcs#introduction-to-dns-services-api)
 - [Private DNS Cloud Docs](https://cloud.ibm.com/docs/dns-svcs?topic=dns-svcs-about-dns-services)