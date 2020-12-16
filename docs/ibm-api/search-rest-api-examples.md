---
layout: default
title: Search API Examples
parent: IBM Cloud Rest API
nav_order: 1
---

# Overview
This page shows examples of interacting with the IBM Cloud REST [Search API](https://cloud.ibm.com/apidocs/search).  

## Prerequisites
 - You have a valid IAM Token as outlined [here](./index.md)

## List all supported resource types
Retrieves a list of all the resource types supported by the Cloud and Classic infrastructure global catalog.

```shell
curl -s -X GET https://api.global-search-tagging.cloud.ibm.com/v2/resources/supported_types \ 
-H "authorization: ${iam_token}" | jq -r
{
  "supported_types": [
    "resource-instance",
    "resource-instance-provision-behind",
    "resource-group",
    "resource-binding",
    "resource-alias",
    "k8-cluster",
    "k8-location",
    "is-vpc",
    "is-instance",
    "is-key",
    "is-volume",
    "is-load-balancer",
    "is-vpn",
    "is-image",
    "is-security-group",
    "is-subnet",
    "is-public-gateway",
    "is-floating-ip",
    "is-network-acl",
    "is-endpoint-gateway",
    "is-flow-log-collector",
    "is-instance-group",
    "is-dedicated-host",
    "is-family",
    "cf-user-provided-service-instance",
    "cf-space",
    "cf-service-instance",
    "cf-service-binding",
    "cf-organization",
    "cf-application",
    "vmware-solutions",
    "ims-block-storage",
    "ims-cloud-backup",
    "ims-cloud-object-storage-infrastructure",
    "ims-file-storage",
    "ims-cdn-powered-by-akamai",
    "ims-direct-link-cloud-connect",
    "ims-direct-link-cloud-exchange",
    "ims-direct-link-colocation",
    "ims-direct-link-network-service-provider",
    "ims-hardware-firewall",
    "ims-hardware-firewall-dedicated",
    "ims-fortigate-security-appliance-1gb",
    "ims-fortigate-security-appliance-10gb",
    "ims-network-gateway-juniper-vsrx",
    "ims-network-gateway-byoa",
    "ims-virtual-router-appliance-copy",
    "ims-ibm-cloud-load-balancer",
    "ims-bare-metal",
    "ims-virtual-server",
    "ims-dedicated-host",
    "ims-image",
    "activity-insights",
    "network-insights",
    "security-advisor"
  ]
}
```

## Searching IBM Cloud Classic Resources
The following examples show how to search against Classic Infrastructure (IaaS, SoftLayer) resources on your IBM Cloud account. 

### Find Classic IaaS virtual instances by tag:
In order to search for resources on your account you need to supply your Account ID. From a terminal you can run the command `ibmcloud account show` to grab the Account ID. 

```shell
$ export ACCOUNT_ID=<Your IBM Account ID>
```

In this example I am searching for all IaaS instances with the tag `consul`

```shell
$ curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \
-H "content-type: application/json" -H "accept: application/json" -H \
"Authorization: ${iam_token}" -d '{"query":"(type:virtual-server AND family:ims) AND (tags:consul)"}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "consul-server3.cdetesting.com",
      "type": "virtual-server",
      "family": "ims",
      "crn": "crn:v1:bluemix:public:virtual-server:wdc06:a/xxxxxxxxxxxxxxxxxxxxxx:90725760::"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "consul-server2.cdetesting.com",
      "type": "virtual-server",
      "family": "ims",
      "crn": "crn:v1:bluemix:public:virtual-server:wdc06:a/xxxxxxxxxxxxxxxxxxxxxx:90725768::"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "consul-server1.cdetesting.com",
      "type": "virtual-server",
      "family": "ims",
      "crn": "crn:v1:bluemix:public:virtual-server:wdc06:a/xxxxxxxxxxxxxxxxxxxxxx:90725770::"
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxx"
}
```

### Find Block storage volumes with specific note
The Classic File and Block volumes do not support tagging via the Cloud Portal. In their place their is a field to store notes about the volume. We will run a search for all block volumes with the note `ryantiffany` and just return their names. You can tag these volumes via the Tagging API. For an example of how to tag Block / File storage see the [following example](https://gist.github.com/greyhoundforty/1cd71970a7d7af376e64719d7b3bd20a).

```shell
$ curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?limit=50" \
-H "accept: application/json" -H "Authorization: ${iam_token}" -H "content-type: application/json" \
-d '{"query":"(type:block-storage AND service_name:block-storage) AND (doc.notes:ryantiffany)" , "fields": [ "name", "tags" ]}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "name": "SL02SEL78003-13",
      "crn": "crn:v1:bluemix:public:block-storage:wdc07:a/xxxxxxxxxxxxxxxxxxxxxx:53093855::",
      "tags": []
    }
  ],
  "limit": 50,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```

### Find all VMware Solutions deployments
The following call with return all the [VMware Dedicated](https://cloud.ibm.com/docs/vmwaresolutions?topic=vmwaresolutions-vc_vcenterserveroverview) and [VMware Shared](https://cloud.ibm.com/docs/vmwaresolutions?topic=vmwaresolutions-shared_overview) deployments on the account.


```shell
curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \ 
-H "accept: application/json" -H "Authorization: ${iam_token}" -H "content-type: application/json" \ 
-d '{"query":"type:vmware-solutions"}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "vcs-rt",
      "type": "vmware-solutions",
      "family": "vmware",
      "crn": "crn:v1:bluemix:public:vmware-solutions:global:a/xxxxxxxxxxxxxxxxxxxxxx:55c17e7d-eaed-4b4f-xxxx-xxxxxxxx::"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "primary-rt",
      "type": "vmware-solutions",
      "family": "vmware",
      "crn": "crn:v1:bluemix:public:vmware-solutions:global:a/xxxxxxxxxxxxxxxxxxxxxx:707319d9-2f6b-4b48-xxxx-xxxxxxxx::"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "jb-test-1",
      "type": "vmware-solutions",
      "family": "vmware",
      "crn": "crn:v1:bluemix:public:vmware-solutions:global:a/xxxxxxxxxxxxxxxxxxxxxx:824912d4-10be-4421-xxxx-xxxxxxxx::"
    },
    {
      "account_id": "6c27214690345bfb75bb1f2b28a20504",
      "name": "centerpoint",
      "type": "vmware-solutions",
      "family": "vmware",
      "crn": "crn:v1:bluemix:public:vmware-solutions:global:a/6c27214690345bfb75bb1f2b28a20504:900d7d94-8236-4382-xxxx-xxxxxxxx:"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "vcd-rt",
      "type": "vmware-solutions",
      "family": "vmware",
      "crn": "crn:v1:bluemix:public:vmware-solutions:global:a/xxxxxxxxxxxxxxxxxxxxxx:9ac98b37-cd31-4800-xxxx-xxxxxxxx::"
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```

### Find all Juniper Network Gateway appliances in the Washington 7 Datacenter

```shell
curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \ 
-H "accept: application/json" -H "Authorization: ${iam_token}" -H "content-type: application/json" \ 
-d '{"query":"(family:ims AND type:network-gateway-juniper-vsrx) AND (region:washington-7)"}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "ha-srx",
      "type": "network-gateway-juniper-vsrx",
      "family": "ims",
      "crn": "crn:v1:bluemix:public:network-gateway-juniper-vsrx:washington-7:a/xxxxxxxxxxxxxxxxxxxxxx:538282::"
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```

## Cloud Resources

### Find all Kubernetes Clusters and just return their name and associated tags:
**Note**: the CRN is always returned even if not part of the field return parameters.

```shell
curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \ 
-H "content-type: application/json" -H "accept: application/json" -H "Authorization: ${iam_token}" \ 
-d '{"query": "type:k8-cluster", "fields": [ "name", "tags", "service_instance" ]}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "name": "devcluster",
      "service_instance": "bpdhxxxx",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:bpdhxxxx::",
      "tags": [
        "ryantiffany"
      ]
    },
    {
      "name": "mycluster-dal10-b3c.4x16",
      "service_instance": "bsdexxxx",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:bsdexxxx::",
      "tags": [
        "jack"
      ]
    },
    {
      "name": "aamir-testcluster-sjc04-b3c.4x16",
      "service_instance": "bslaxxxx",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:bslaxxxx::",
      "tags": []
    },
    {
      "name": "cluster2",
      "service_instance": "btckxxxx",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:btckxxxx::",
      "tags": []
    },
    {
      "name": "Russell_Cluster",
      "service_instance": "bthrxxxx",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:bthrxxxx::",
      "tags": []
    },
    {
      "name": "rt-us-south",
      "service_instance": "btr2xxxx",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:btr2xxxx::",
      "tags": [
        "region:us-south",
        "ryantiffany"
      ]
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```

## Find all Kubernetes clusters in a given region
In this example we are searching for all Kubernetes Clusters in the Tokyo region:

```shell
$ curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \ 
-H "accept: application/json" -H "Authorization: ${iam_token}" -H "content-type: application/json" \ 
-d '{"query":"type:k8-cluster AND region:jp-tok"}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "subredOpen",
      "type": "k8-cluster",
      "family": "containers",
      "crn": "crn:v1:bluemix:public:containers-kubernetes:jp-tok:a/xxxxxxxxxxxxxxxxxxxxxx:blr1xxxxxx:"
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```

### Find all VPC instances in a specific region with specific tags
Since VPC instances stretch across multiple zones in a region we use the wildcard search term `region:us-east*` in this case to show all instances within the US-East VPC region.

```shell
$ curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \ 
-H "accept: application/json" -H "Authorization: ${iam_token}" -H "content-type: application/json" \ 
-d '{"query":"(family:is AND type:instance AND region:us-east*)"}'
```

**Example output**
```json
{
  "items": [
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "wgtest-z1-dev",
      "type": "instance",
      "family": "is",
      "crn": "crn:v1:bluemix:public:is:us-east-1:a/xxxxxxxxxxxxxxxxxxxxxx::instance:0757_2834694d-ad33-4b81-xxxx-xxxxxxxx"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "wgtest-wg-dev",
      "type": "instance",
      "family": "is",
      "crn": "crn:v1:bluemix:public:is:us-east-1:a/xxxxxxxxxxxxxxxxxxxxxx::instance:0757_ebdfe595-c96b-4924-xxxx-xxxxxxxx"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "vpntest",
      "type": "instance",
      "family": "is",
      "crn": "crn:v1:bluemix:public:is:us-east-1:a/xxxxxxxxxxxxxxxxxxxxxx::instance:0757_f89fa662-71ab-45e1-xxxx-xxxxxxxx"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "wgtest-z2-dev",
      "type": "instance",
      "family": "is",
      "crn": "crn:v1:bluemix:public:is:us-east-2:a/xxxxxxxxxxxxxxxxxxxxxx::instance:0767_65809d72-adca-4c5d-xxxx-xxxxxxxx"
    },
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "wgtest-z3-dev",
      "type": "instance",
      "family": "is",
      "crn": "crn:v1:bluemix:public:is:us-east-3:a/xxxxxxxxxxxxxxxxxxxxxx::instance:0777_e7321492-179b-4f26-xxxx-xxxxxxxx"
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```

### Find all Database for Etcd instances with a specific tag

```shell
curl -s -X POST "https://api.global-search-tagging.cloud.ibm.com/v3/resources/search?account_id=${ACCOUNT_ID}" \ 
-H "accept: application/json" -H "Authorization: ${iam_token}" -H "content-type: application/json" \ 
-d '{"query":"(type:resource-instance AND service_name:databases-for-etcd) AND (tags:ryantiffany)"}' | jq -r
```

**Example Output**
```shell
{
  "items": [
    {
      "account_id": "xxxxxxxxxxxxxxxxxxxxxx",
      "name": "rt-1978903720bf",
      "type": "resource-instance",
      "family": "resource_controller",
      "crn": "crn:v1:bluemix:public:databases-for-etcd:us-south:a/xxxxxxxxxxxxxxxxxxxxxx:41625c25-f836-491e-xxx-xxxxxx::"
    }
  ],
  "limit": 10,
  "search_cursor": "xxxxxxxxxxxxxxxxxxxxxx"
}
```