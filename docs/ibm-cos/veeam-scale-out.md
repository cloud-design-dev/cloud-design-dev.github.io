---
layout: default
title: Scale Out Repository with Veeam and IBM COS
parent: IBM Cloud Object Storage
nav_order: 1
---

# Scale Out Veeam Storage with IBM Cloud Object Storage

## Prerequisites

* IBM Cloud Object Storage bucket. See [this guide](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage#gs-create-buckets) if you need to create an Object Storage Bucket.
* Veeam Backup Server with an un-used local or Block volume drive.

### Steps

1. [IBM Cloud Pre-Work](ibm-cloud-pre-work.md)
2. [Prepare Windows server drive to be used with Veeam](prepare-windows-server-drive.md)
3. [Create new Backup Repository in Veeam](create-new-veeam-backup-repository.md)
4. [Add IBM Cloud Object Storage as a Scale-Out Repository in Veeam](add-scale-out-storage-to-veeam.md) 

### Links

* [IBM Cloud Object Storage](https://www.ibm.com/cloud/object-storage)
* [Veeam Scale Out Repository](https://helpcenter.veeam.com/docs/backup/vsphere/backup_repository_sobr.html?ver=100)
* [Veeam Backup and Replication](https://www.veeam.com/vm-backup-recovery-replication-software.html?ad=menu-products)

## IBM Cloud Pre-Work  
### Launch IBM Cloud Shell
Launch [Cloud Shell](https://cloud.ibm.com/docs/cloud-shell?topic=cloud-shell-getting-started) to run through the following steps. The icon can be found in the upper right navigation bar in the cloud portal.

![Launch Cloud Shell](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-11-02-05.png)

### Finding Your Cloud Object Storage Instance ID:
We will use the _resource service-instance_ operator to get our Cloud Object Storage instance id \(GUID\). Use the handy [jq](https://stedolan.github.io/jq/) utility to only return the GUID.

```bash
$ ibmcloud resource service-instance <name-of-icos-instance> --output json | jq -r '.[].guid'
```

> You will use the value returned by the above command in another command. Please note it.

#### Create a Service ID

We will create a service ID to interact with our ICOS bucket. A service ID identifies a service or application similar to how a user ID identifies a user. We can assign specific access policies to the service ID that restrict permissions for using specific services.

```shell
ibmcloud iam service-id-create <name-of-your-service-id> --description "Service ID for Veeam Scale out repository" --output json
```

This command simply returns the ID of the service-id that was created. This will be used in place of  in the next command.

```shell
ibmcloud iam service-id <name-of-your-service-id> --output json | jq -r '.[].id'
```

> You will use the value returned by the above command in another command. Please note it.

#### Assign Service ID an access policy

We will now create an access policy that gives the _Service ID_ write access to a single IBM Cloud Object Storage bucket.

```shell
ibmcloud iam service-policy-create <Service ID> --roles Writer --service-name cloud-object-storage --service-instance <Service Instance GUID> --resource-type bucket --resource <bucket-name>
```

#### Create Service Credentials for the Service ID

Veeam uses HMAC credentials \(Secret Key/Access Key\) so we will need to generate some from cloud shell. We will be binding these new credentials to our Service ID

```shell
$ ibmcloud resource service-key-create <name of service key> Writer --instance-id <Service Instance GUID> --service-id <Service ID> --parameters '{"HMAC":true}'
```

> Save the **access\_key\_id** and **secret\_access\_key** as we will need these when we configure the scale out repository in Veeam.
