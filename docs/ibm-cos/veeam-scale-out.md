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

1. [IBM Cloud Pre-Work](#ibm-cloud-pre-work)
2. [Prepare Windows server drive to be used with Veeam](#prepare-windows-server-drive)
3. [Create new Backup Repository in Veeam](#create-new-veeam-backup-repository)
4. [Add IBM Cloud Object Storage as a Scale-Out Repository in Veeam](#add-scale-out-storage-to-veeam) 

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

**Note:** You will use the value returned by the above command in another command. Please save it.

#### Create a Service ID  
We will create a service ID to interact with our ICOS bucket. A service ID identifies a service or application similar to how a user ID identifies a user. We can assign specific access policies to the service ID that restrict permissions for using specific services.

```shell
ibmcloud iam service-id-create <name-of-your-service-id> --description "Service ID for Veeam Scale out repository" --output json
```

This command simply returns the ID of the service-id that was created. This will be used in place of  in the next command.

```shell
ibmcloud iam service-id <name-of-your-service-id> --output json | jq -r '.[].id'
```

**Note:** You will use the value returned by the above command in another command. Please save it.

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

**Note:** Save the **access\_key\_id** and **secret\_access\_key** as we will need these when we configure the scale out repository in Veeam.

## Prepare Windows Server Drive
Upon initial login to Windows server you should see Server Manager. If not then launch Server Manager and from the left hand navigation select:

* **File and Storage Services**
* **Disks**
* **The drive we want to use as our new Veeam Backup Repository**

![Select second drive](https://dsc.cloud/quickshare/select-second-drive.png)

In the Volumes view click on **Tasks** and select _New Volume_.  
![New Volume](https://dsc.cloud/quickshare/volume-tasks-new-volume.png)

This will start the new Volume Wizard. Under _Server and Disk_ select your local server and the drive where this volume will be created. Click **Next**  
![Select server and drive](https://dsc.cloud/quickshare/server-and-disks.png)

For both the **Size** and **Drive Letter** selections you can simply click Next as we are leaving these as default.

When it comes the filesystem we will want to select _ReFS_ for the _File System_ and _64K_ for the _Allocation unit size_ and then click **Next**. After you have reviewed the volume creation details click **Create** to provision the volume.  
![File System Settings](https://dsc.cloud/quickshare/file-system-settings.png)

The drive is now ready to be used with Veeam.

## Create New Veeam Backup Repository
From the Veeam Backup and Replication console click **Backup Infrastructure** and then right click on **Backup Repositories** and select **Add backup repository** to launch the Repository creation wizard.

![Add backup repository wizard](https://dsc.cloud/quickshare/add-backup-repository.png)

Select _Direct Attached Storage_ in the first dialog box and _Microsoft Windows_ in the second. Give the backup repository a name and click **Next**  
![](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-09-50-40.png)

Select your local server and click Populate to list locally attached available drives. In my case I select `F` and select **Next**  
![Select Repository Drive](https://dsc.cloud/quickshare/select-repo-drive.png)

For both the _Repository_ and _Mount Server_ dialog boxes you can accept the defaults and just click **Next**

On the review dialog box make sure to leave _Import existing backups automatically_ unchecked since we're starting off with a fresh set of backup job. Once you're satisfied everything looks correct click **Apply**.  
![Create repository](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-09-55-55.png)

**Note:** When you see a dialog box asking _Change the configuration backup location to the newly created repository?_ select **No**. If you select **Yes** the backup repository becomes un-usable for Scale out ICOS storage.

## Add Scale Out Storage to Veeam

From the Veeam Backup and Replication console click on **Backup Infrastructure** and then right click on **Add Scale out repository**:

![Add Scale Out](https://dsc.cloud/quickshare/add-scale-out.png)

Give the new Scale out repository a name and click **Next**

. ![Name repository](https://dsc.cloud/quickshare/name-scale-out-repo.png)

Under Performance Tier click **Add** and then select the newly created backup repository.  
![Add backup repository to scale out](https://dsc.cloud/quickshare/add-repo-to-scale-out.png)

Select _Extend scale-out with object storage_ and click the **Add** button. ![x](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-10-06-01.png)

Select IBM Cloud Object Storage

 ![x](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-10-07-19.png)

Add the Cloud Object Storage endpoint for your bucket as well as the HMAC credentials we created earlier. The bucket that I am going to be offloading backups to was created as a us-south regional bucket. As such I am adding the private us-south endpoint `s3.private.us-south.cloud-object-storage.appdomain.cloud`. To see all available endpoints see [Object Storage Endpoints](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-endpoints).

![Add endpoint](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-10-09-49.png)

Next to credentials click Add. This is where you will add the HMAC keys that we generated previously. With those credentials added click Next. You will now select the COS bucket to use. Under Folder selection click **Browse** and click **Add Folder** to create a new directory within the bucket. Click **Next** to get to the Review page. If everything looks good click **Finish**.

You are now dropped back in to the Scale out repository wizard. You can now set the **age out** policy for backups in this backup repository. Make sure the _Move backups to object storage_ checkbox is checked and set your policy. The default is 30 days.

![](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-10-26-45.png)

Click **Apply** and review the creation details. Click **Finish**.

Make sure to select your newly created backup repository when creating new backup jobs to ensure that your backup roll-off operations run as expected.

![Targeting new backup repository](https://dsc.cloud/quickshare/Shared-Image-2020-07-24-10-34-11.png)
