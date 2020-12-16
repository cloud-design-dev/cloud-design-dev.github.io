---
layout: default
title: Cross account Object Storage bucket sync with free IKS Cluster
parent: Kubernetes
nav_order: 2
---

# Overview
In this guide I will show you how to sync ICOS bucket objects between accounts using Kubernetes. You can spin up 1 free IKS cluster on your account which will be automatically deleted after 30 days. 

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
* [Create free IBM Cloud Kubernetes cluster](#create-free-ibm-cloud-kubernetes-cluster)
* [Create Kubernetes secret for use with job](#create-kubernetes-secret-for-use-with-job)
* [Download Kubernetes Job file](#download-kubernetes-job-file)

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
Save the **access_key_id** and **secret_access_key** as we will be using these with our Kubernetes job. 

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
Save the **access_key_id** and **secret_access_key** as we will be using these with our Kubernetes job. 

## Create free IBM Cloud Kubernetes cluster
You can have 1 free cluster at a time in IBM Cloud Kubernetes Service, and each free cluster expires in 30 days. This will allow is to run the Minio sync as a Kubernetes Job. 

```shell 
$ ibmcloud ks cluster create classic --name <name of cluster>
```

The cluster should be ready in about 10-15 minutes. You can verify the status of the creation by running the `ibmcloud ks clusters` and seeing when State returns as *normal*:

```shell
ryan@cloudshell:~$ ibmcloud ks clusters 
OK
Name             ID                     State    Created          Workers   Location   Version       Resource Group Name   Provider   
freeikscluster   bstcs2nf0bkrt2968nm0   normal   14 minutes ago   1         par01      1.17.9_1534   default               classic
```

## Create Kubernetes secret for use with job 
In order for our container to speak to the Cloud Object storage buckets we'll need to specify some environmental variables to be stored as [Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/). The first step is to target our free cluster:

```shell
$ ibmcloud ks cluster config --cluster <name of cluster>
```

With `kubectl` now configured to talk to our cluster we can create our Kubernetes secrets. 

```shell
$ kubectl create secret generic <name of secret> --from-literal SOURCE_ACCESS_KEY=VALUE --from-literal SOURCE_SECRET_KEY=VALUE --from-literal SOURCE_REGION=VALUE --from-literal SOURCE_BUCKET=VALUE --from-literal DESTINATION_ACCESS_KEY=VALUE --from-literal DESTINATION_SECRET_KEY=VALUE --from-literal DESTINATION_REGION=VALUE --from-literal DESTINATION_BUCKET=VALUE
```

![Create Kubernetes Secret](https://dsc.cloud/quickshare/iks-create-secret.png)

## Download Kubernetes Job file
We specify our Kubernetes job in a yaml file hosted [here](https://gist.github.com/greyhoundforty/b26d66b33f97d8368e3dd7869b7bbc5e) on Github. 

```shell
$ wget https://gist.githubusercontent.com/greyhoundforty/b26d66b33f97d8368e3dd7869b7bbc5e/raw/926e7088cff7c0a991310065832d646e513890f1/job.yaml
```

With the Job spec downloaded we can now submit the job to Kubernetes

```shell
$ kubectl create -f job.yaml
```

In my testing I am only syncing a few times so by the time I check the Kubernetes pods, the job has already completed. Looking at the logs I am able to verify that the contents have been synced between the Object Storage buckets.

```shell
ryan@cloudshell:~$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
mc-icos-sync-test-c9zkq   0/1     Completed   0          102s
```

![Show pod logs](https://dsc.cloud/quickshare/get-pod-logs.png)

