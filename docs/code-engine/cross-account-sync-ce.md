---
layout: default
title: Cross account Object Storage bucket sync with Code Engine
parent: IBM Code Engine
nav_order: 1
---

# Overview
In this guide I will show you how to sync ICOS bucket objects between accounts using [Code Engine](https://cloud.ibm.com/docs/codeengine). Code Engine provides a platform to unify the deployment of all of your container-based applications on a Kubernetes-based infrastructure. The Code Engine experience is designed so that you can focus on writing code without the need for you to learn, or even know about, Kubernetes.

> Code Engine is currently an experimental offering and all resources are deleted every 7 days.

## Steps
* [Preparing Accounts](#preparing-accounts)
+ [Source Account](#source-account)
    - [Create Service ID](#create-service-id)
    - [Create Reader access policy for newly created service id](#create-reader-access-policy-for-newly-created-service-id)
    - [Generate HMAC credentials tied to our service ID](#generate-hmac-credentials-tied-to-our-service-id)
+ [Destination Account](#destination-account)
    - [Create Service ID](#create-service-id-1)
    - [Create Reader access policy for newly created service id](#create-reader-access-policy-for-newly-created-service-id-1)
    - [Generate HMAC credentials tied to our service ID](#generate-hmac-credentials-tied-to-our-service-id-1)
* [Create Code Engine Project via Cloud Shell](#create-code-engine-project-via-cloud-shell)
* [Create Code Engine Secrets](#create-code-engine-secrets)
* [Create Code Engine Project Job definition with environmental variables](#create-code-engine-project-job-definition-with-environmental-variables)
* [Submit Code Engine Job](#submit-code-engine-job)

## Preparing Accounts
We will be using Cloud Shell to generate Service IDs and Object Storage credentials for both the source and destination accounts. 

### Source Account 
We will create a service ID on the source account. A service ID identifies a service or application similar to how a user ID identifies a user. We can assign specific access policies to the service ID that restrict permissions for using specific services: in this case it gets read-only access to an IBM Cloud Object Storage bucket. 

#### Create Service ID
```shell
$ ibmcloud iam service-id-create <name-of-your-service-id> --description "Service ID for read-only access to bucket" --output json
```

![Service ID Creation](https://dsc.cloud/quickshare/source-service-id.png)

#### Create Reader access policy for newly created service id
Now we will limit the scope of this service ID to have read only access to our source Object Storage bucket. 

```shell
$ ibmcloud iam service-policy-create <Service ID> --roles Reader --service-name cloud-object-storage --service-instance <Service Instance GUID> --resource-type bucket --resource <bucket-name>
```

*Service Instance GUID*  - This is the GUID of the Cloud Object Storage instance. You can retrieve this with the command: `ibmcloud resource service-instance <name of icos instance>`

![Expected Output Example](https://dsc.cloud/quickshare/create-source-service-policy.png)

#### Generate HMAC credentials tied to our service ID 
In order for the Minio client to talk to each Object Storage instance it will need HMAC credentials (Access Key and Secret Key in S3 parlance). 

```shell
$ ibmcloud resource service-key-create source-icos-service-creds Reader --instance-id <Service Instance GUID> --service-id <Service ID> --parameters '{"HMAC":true}'
```
Save the **access_key_id** and **secret_access_key** as we will be using these in our Code Engine project. 

![Create HMAC Credentials](https://dsc.cloud/quickshare/source-hmac-credentials.png)

---------------------------------------------------------------

### Destination Account
We will create a service ID on the destination account. A service ID identifies a service or application similar to how a user ID identifies a user. We can assign specific access policies to the service ID that restrict permissions for using specific services: in this case it gets write access to an IBM Cloud Object Storage bucket.  

#### Create Service ID
```shell
$ ibmcloud  iam service-id-create <name-of-your-service-id> --description "Service ID for write access to bucket" --output json
```

![Expected Output Example](https://dsc.cloud/quickshare/destination-service-id.png)

#### Create Reader access policy for newly created service id
Now we will limit the scope of this service ID to have read only access to our source Object Storage bucket. 

```shell 
$ ibmcloud iam service-policy-create <Service ID> --roles Writer --service-name cloud-object-storage --service-instance <Service Instance GUID> --resource-type bucket --resource <bucket-name>
```

*Service Instance GUID*  - This is the GUID of the Cloud Object Storage instance. You can retrieve this with the command: `ibmcloud resource service-instance <name of icos instance>`

#### Generate HMAC credentials tied to our service ID 
We'll follow the same procedure as last time to generate the HMAC credentials, but this time on the destination account.

```shell
$ ibmcloud resource service-key-create destination-icos-service-creds Writer --instance-id <Service Instance GUID> --service-id <Service ID> --parameters '{"HMAC":true}'
```
Save the **access_key_id** and **secret_access_key** as we will be using these in with our Code Engine project. 

## Create Code Engine Project via Cloud Shell
In order to create our Code Engine project we need to make sure that our cloud shell session is targeting the correct resource group. You can do this by using the `target -g` option with the IBM Cloud CLI. 

```shell
$ ibmcloud target -g <Resource Group>
```
With the correct Resource Group set, we can now create our Code Engine project. We add the `--target` flag to ensure that future Code Engine commands are targeting the correct project.

```
$ ibmcloud ce project create -n <project_name> --target
```

![Create Code Engine Project](https://dsc.cloud/quickshare/ce-create-project.png)

## Create Code Engine Secrets
In order for our minio powered container to sync objects between the accounts it needs access to the Access and Secret keys we created earlier. We will use the `secret create` option to store all of values in a single secret that we can then reference in our job definition.

 - SOURCE_ACCESS_KEY: Access Key generated on Source account
 - SOURCE_SECRET_KEY: Secret Key generated on Source account
 - SOURCE_REGION: Cloud Object Storage endpoint for the Source bucket
 - SOURCE_BUCKET: Name of bucket on Source account
 - DESTINATION_ACCESS_KEY: Access Key generated on Destination account
 - DESTINATION_SECRET_KEY: Secret Key generated on Destination account
 - DESTINATION_REGION: Cloud Object Storage endpoint for the Destination bucket
 - DESTINATION_BUCKET: Name of bucket on Destination account

```shell
$ ibmcloud ce secret create --name ce-sync-secret --from-literal SOURCE_ACCESS_KEY=VALUE --from-literal SOURCE_SECRET_KEY=VALUE --from-literal SOURCE_REGION=VALUE --from-literal SOURCE_BUCKET=VALUE --from-literal DESTINATION_ACCESS_KEY=VALUE --from-literal DESTINATION_SECRET_KEY=VALUE --from-literal DESTINATION_REGION=VALUE --from-literal DESTINATION_BUCKET=VALUE
```

![Create Code Engine Secret](https://dsc.cloud/quickshare/ce-create-secret.png)

## Create Code Engine Project Job definition with environmental variables
Now that our project has been created we need to create our Job definition. In Code Engine terms a job is a stand-alone executable for batch jobs. Unlike applications, which react to incoming HTTP requests, jobs are meant to be used for running container images that contain an executable that is designed to run one time and then exit.

```shell
$ ibmcloud ce jobdef create --name JOBDEF_NAME --image IMAGE_REF --env-from-secret SECRET_NAME 
```

![Create Job Definition](https://dsc.cloud/quickshare/ce-create-jobdef.png)

You can view the definition of the job using the command `ibmcloud ce jobdef get -n <name of job definition>`

```shell
$ ibmcloud ce jobdef get -n ce-mc-sync-jobdef
Project 'ce-minio-sync' and all its contents will be automatically deleted 7 days from now.
Getting job definition 'ce-mc-sync-jobdef'...
Name:        ce-mc-sync-jobdef  
Project ID:  1d7514d2-ce89  
Metadata:    
  Creation Timestamp:  2020-08-17 14:51:00 +0000 UTC  
  Generation:          1  
  Resource Version:    223595401  
  Self Link:           /apis/codeengine.cloud.ibm.com/v1alpha1/namespaces/1d7514d2-ce89/jobdefinitions/ce-mc-sync-jobdef  
  UID:                 0fb11f71-a912-44f9-88e3-1d1612f8e8ab  
Spec:        
  Containers:  
    Image:  greyhoundforty/icos-ce-sync:1  
    Name:   ce-mc-sync-jobdef  
    Commands:  
    Arguments:  
    Env:    
      Name:  SOURCE_BUCKET  
      Value From Secret Key Ref:  
        Key:   SOURCE_BUCKET  
        Name:  ce-sync-secret  
    Env:    
      Name:  SOURCE_REGION  
      Value From Secret Key Ref:  
        Key:   SOURCE_REGION  
        Name:  ce-sync-secret  
    Env:    
      Name:  SOURCE_SECRET_KEY  
      Value From Secret Key Ref:  
        Key:   SOURCE_SECRET_KEY  
        Name:  ce-sync-secret  
    Env:    
      Name:  DESTINATION_ACCESS_KEY  
      Value From Secret Key Ref:  
        Key:   DESTINATION_ACCESS_KEY  
        Name:  ce-sync-secret  
    Env:    
      Name:  DESTINATION_BUCKET  
      Value From Secret Key Ref:  
        Key:   DESTINATION_BUCKET  
        Name:  ce-sync-secret  
    Env:    
      Name:  DESTINATION_REGION  
      Value From Secret Key Ref:  
        Key:   DESTINATION_REGION  
        Name:  ce-sync-secret  
    Env:    
      Name:  DESTINATION_SECRET_KEY  
      Value From Secret Key Ref:  
        Key:   DESTINATION_SECRET_KEY  
        Name:  ce-sync-secret  
    Env:    
      Name:  SOURCE_ACCESS_KEY  
      Value From Secret Key Ref:  
        Key:   SOURCE_ACCESS_KEY  
        Name:  ce-sync-secret  
    Resource Requests:  
      Cpu:     1  
      Memory:  128Mi  
OK
```

## Submit Code Engine Job
It is now time to submit our Job to Code Engine. The maximum time a job can run is 10 hours, but in most cases ICOS syncing takes significantly less time to complete. 

```shell
$ ibmcloud ce job run --name <name of job> --jobdef <name of job definition>
```

In my testing I am only syncing a few times so by the time I check the Kubernetes pods, the job has already completed. Looking at the logs I am able to verify that the contents have been synced between the Object Storage buckets.

```shell
ryan@cloudshell:~$ ibmcloud ce job run --name  ce-mc-sync-jobv1 --jobdef ce-mc-sync-jobdef
Project 'ce-minio-sync' and all its contents will be automatically deleted 7 days from now.
Creating job 'ce-mc-sync-jobv1'...
OK

ryan@cloudshell:~$ ibmcloud ce project target -n ce-minio-sync --kubecfg
Targeting project 'ce-minio-sync'...
Added context for 'ce-minio-sync' to the current kubeconfig file.
OK
Now targeting environment 'ce-minio-sync'.

ryan@cloudshell:~$ kubectl get pods 
NAME                   READY   STATUS      RESTARTS   AGE
ce-mc-sync-jobv1-0-0   0/1     Completed   0          53s

ryan@cloudshell:~$ ibmcloud ce job list
Project 'ce-minio-sync' and all its contents will be automatically deleted 7 days from now.
Listing jobs...
Name               Age   
ce-mc-sync-jobv1   2m13s   
OK
Command 'job list' performed successfully

ryan@cloudshell:~$ ibmcloud ce job logs -n ce-mc-sync-jobv1
Project 'ce-minio-sync' and all its contents will be automatically deleted 7 days from now.
Logging job 'ce-mc-sync-jobv1' on pod '0'...
Added `source_acct` successfully.
Added `destination_acct` successfully.
`source_acct/wandering-thunder-68-source/as-policy.png` -> `destination_acct/sparkling-sky-47-destination/as-policy.png`
`source_acct/wandering-thunder-68-source/create-source-service-policy.png` -> `destination_acct/sparkling-sky-47-destination/create-source-service-policy.png`
`source_acct/wandering-thunder-68-source/add-backup-repository.png` -> `destination_acct/sparkling-sky-47-destination/add-backup-repository.png`
`source_acct/wandering-thunder-68-source/direct-link-standard.png` -> `destination_acct/sparkling-sky-47-destination/direct-link-standard.png`
`source_acct/wandering-thunder-68-source/create-workspace.png` -> `destination_acct/sparkling-sky-47-destination/create-workspace.png`
`source_acct/wandering-thunder-68-source/direct-link-byoip.png` -> `destination_acct/sparkling-sky-47-destination/direct-link-byoip.png`
`source_acct/wandering-thunder-68-source/add-scale-out.png` -> `destination_acct/sparkling-sky-47-destination/add-scale-out.png`
`source_acct/wandering-thunder-68-source/filter-vpc.png` -> `destination_acct/sparkling-sky-47-destination/filter-vpc.png`
`source_acct/wandering-thunder-68-source/k8s-storage.png` -> `destination_acct/sparkling-sky-47-destination/k8s-storage.png`
`source_acct/wandering-thunder-68-source/Picture1.png` -> `destination_acct/sparkling-sky-47-destination/Picture1.png`
`source_acct/wandering-thunder-68-source/iks-storage-Page-1.png` -> `destination_acct/sparkling-sky-47-destination/iks-storage-Page-1.png`
`source_acct/wandering-thunder-68-source/dl-copy.png` -> `destination_acct/sparkling-sky-47-destination/dl-copy.png`
`source_acct/wandering-thunder-68-source/rt-us-east.gv.png` -> `destination_acct/sparkling-sky-47-destination/rt-us-east.gv.png`
`source_acct/wandering-thunder-68-source/ns-secrets-cm.png` -> `destination_acct/sparkling-sky-47-destination/ns-secrets-cm.png`
Total: 0 B, Transferred: 2.30 MiB, Speed: 1.66 MiB/s

OK
Command 'job logs' performed successfully
```

If you need to sync contents from the source bucket to the destination bucket again, simply run another job (with a new name) and Code Engine will take care of it for you. 