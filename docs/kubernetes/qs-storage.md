---
layout: default
title: Using QuantaStor as a Persistent Volume Provider for Kubernetes
parent: Kubernetes
nav_order: 2
---

- [Overview](#overview)
  - [IBM Cloud Kubernetes Service (IKS)](#ibm-cloud-kubernetes-service-iks)
    - [Single Zone Clusters](#single-zone-clusters)
    - [Multizone Clusters](#multizone-clusters)
  - [QuantaStor](#quantastor)
  - [Prerequisites](#prerequisites)
  - [Steps](#steps)
    - [Use the slcli to get the IKS Worker Node IPs](#use-the-slcli-to-get-the-iks-worker-node-ips)
    - [Create Storage Pool in QuantaStor](#create-storage-pool-in-quantastor)
    - [Create QuantaStor Network Share](#create-quantastor-network-share)
    - [Enable NFS access from IKS Worker node IPs](#enable-nfs-access-from-iks-worker-node-ips)
    - [Deploy NFS provisioner Helm chart](#deploy-nfs-provisioner-helm-chart)
    - [Deploy example pod and write to mounted persistent volume](#deploy-example-pod-and-write-to-mounted-persistent-volume)

# Overview
In this guide I will be showing you how to use QuantaStor as a persistent volume provider for Kubernetes on IBM Cloud.

## IBM Cloud Kubernetes Service (IKS)
The IBM Cloud Kubernetes service (IKS) has several options for persistent storage but they can vary greatly depending on the type of cluster that you've provisioned. Within IBM Cloud you can deploy Kubernetes [Singlezone](https://cloud.ibm.com/docs/containers?topic=containers-ha_clusters#single_zone) clusters or [Multizone](https://cloud.ibm.com/docs/containers?topic=containers-ha_clusters#multizone) clusters.

### Single Zone Clusters
For single zone clusters (a cluster whose worker nodes all reside in the same Datacenter) the following options are supported out of the box:
 - [IBM Cloud File Storage](https://cloud.ibm.com/docs/containers?topic=containers-file_storage)
 - [IBM Cloud Block Storage](https://cloud.ibm.com/docs/containers?topic=containers-block_storage) 
 - [IBM Cloud Object Storage](https://cloud.ibm.com/docs/containers?topic=containers-object_storage)

### Multizone Clusters
For multizone clusters (a cluster whose worker nodes span multiple Datacenters in the same region) your options are a little more limited. Both the IBM Cloud File and Block offerings are restricted to the Datacenters in which they are ordered. So if you have a File volume provisioned in Dallas 13, your servers and IKS worker nodes in say Dallas 10 cannot actually interact with that storage. For Multizone clusters the following options are supported out of the box:

-   [IBM Cloud Object Storage](https://cloud.ibm.com/docs/containers?topic=containers-object_storage)    
-   [Portworx](https://cloud.ibm.com/docs/containers?topic=containers-portworx)
-   IBM Cloud Databases - These are fully managed IBM Cloud hosted databases:
    -   [Etcd](https://cloud.ibm.com/catalog/services/databases-for-etcd#about)
    -   [Mongodb](https://cloud.ibm.com/catalog/services/databases-for-mongodb#about)
    -   [Elasticsearch](https://cloud.ibm.com/catalog/services/databases-for-elasticsearch#about)
    -   [Redis](https://cloud.ibm.com/catalog/services/databases-for-redis#about)
    -   [PostgreSQL](https://cloud.ibm.com/catalog/services/databases-for-postgresql#about)
        
[Portworx](https://portworx.com/) is a great option for supporing Multizone persistent storage but you have to also run an instance of Etcd and it is best used with Bare Metal servers which would increase the cost of your cluster and would likely be overkill for smaller dev/test environment.

As an alternative we'll set up a QuantaStor server to act as an NFS Persistent Volume storage provider. Since the storage is from the local Quantastor disks and not the IBM Cloud the volumes can span multiple Datacenters.

## QuantaStor
[QuantaStor Software Defined Storage](https://www.osnexus.com/products/software-defined-storage) supports all major file, block, and object protocols including iSCSI/FC, NFS/SMB, and S3. It features the ability to enable encryption at rest as well as in flight.

## Prerequisites
 - Kubernetes cluster
 - QuantaStor server. You can download the [Community edition](https://www.osnexus.com/downloads) for free. QuantaStor Community Edition keys are capacity limited to 40TB of raw capacity and 4x servers per storage grid.
 - kubectl installed. Guide [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
 - slcli installed. Guide [here](https://softlayer-api-python-client.readthedocs.io/en/latest/install/).   
 - helm installed on your cluster. Guide [here](https://cloud.ibm.com/docs/containers?topic=containers-helm)
    
## Steps
Here are the steps we will be going through in this guide:
 - Use the slcli to get the IKS Worker Node IPs
 - Create Storage Pool in QuantaStor
 - Create QuantaStor Network Share
 - Enable NFS access from IKS Worker node IPs
 - Deploy NFS provisioner Helm chart
 - Deploy example pod and write to mounted persistent volume

### Use the slcli to get the IKS Worker Node IP Ranges
We can gather our worker node IPs using `kubectl` and then use the `slcli` to get the subnet range that will be whitelisted for our NFS share.

```shell
$ kubectl get nodes -o json | jq -r '.items[].metadata.name'
10.184.249.15
10.184.249.2
10.184.249.34
10.221.28.10
10.221.28.37
10.221.28.5

$ slcli --format json subnet lookup 10.184.249.15 | jq -r '.subnet.identifier'
10.184.249.0/26

$ slcli --format json subnet lookup 10.221.28.37 | jq -r '.subnet.identifier'
10.221.28.0/25
```

### Create Storage Pool in QuantaStor
![Create Storage Pool in QuantaStor](https://dsc.cloud/quickshare/CreateStoragePool.png)

### Create QuantaStor Network Share
![Create QuantaStor Network Share 1](https://dsc.cloud/quickshare/CreateNetworkShare1.png)

![Create QuantaStor Network Share 2](https://dsc.cloud/quickshare/CreateNetworkShare2.png)

![Create QuantaStor Network Share 3](https://dsc.cloud/quickshare/NFSExportPath.png)

### Enable NFS access from IKS Worker node IPs
![Enable NFS access from IKS Worker node IPs 1](https://dsc.cloud/quickshare/AddNFSAccess.png)

![Enable NFS access from IKS Worker node IPs 2](https://dsc.cloud/quickshare/AddingNFSAccess.png)

![Enable NFS access from IKS Worker node IPs 3](https://dsc.cloud/quickshare/CompletedAddingNFSAccess.png)

### Deploy NFS provisioner Helm chart
Enter the [nfs-client-provisioner](https://github.com/helm/charts/tree/master/stable/nfs-client-provisioner) helm chart

```shell
$ $ helm install stable/nfs-client-provisioner --set nfs.server=10.171.145.12 --set nfs.path=/export/PVShare --name nfs-client
NAME:   nfs-client
LAST DEPLOYED: Fri Jan 10 09:09:10 2020
NAMESPACE: default
STATUS: DEPLOYED
RESOURCES:
==> v1/ClusterRole
NAME                                      AGE
nfs-client-nfs-client-provisioner-runner  1s

==> v1/ClusterRoleBinding
NAME                                   AGE
run-nfs-client-nfs-client-provisioner  1s

==> v1/Deployment
NAME                               READY  UP-TO-DATE  AVAILABLE  AGE
nfs-client-nfs-client-provisioner  0/1    0           0          1s

==> v1/Role
NAME                                              AGE
leader-locking-nfs-client-nfs-client-provisioner  1s

==> v1/RoleBinding
NAME                                              AGE
leader-locking-nfs-client-nfs-client-provisioner  1s

==> v1/ServiceAccount
NAME                               SECRETS  AGE
nfs-client-nfs-client-provisioner  1        2s

==> v1/StorageClass
NAME        PROVISIONER                                      AGE
nfs-client  cluster.local/nfs-client-nfs-client-provisioner  2s
```

Check that NFS is available as a storageclass.

{% highlight shell %}
$ kubectl get storageclasses | grep nfs-client
nfs-client                   cluster.local/nfs-client-nfs-client-provisioner   104s
{% endhighlight %}

### Deploy example pod and write to mounted persistent volume
Here is an example Persistent Volume Claim yaml file.  
**qs-test-pvc.yml**

{% highlight yaml %}
 apiVersion: v1
 kind: PersistentVolumeClaim
 metadata:
   name: qspvc
 spec:
   storageClassName: nfs-client
   accessModes:
     - ReadWriteMany
   resources:
     requests:
       storage: 2Gi
{% endhighlight %}

**Create pvc**

```shell
$ kubectl create -f qs-test-pvc.yml
persistentvolumeclaim/qspvc created

$ kubectl get persistentvolumeclaim/qspvc
NAME    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
qspvc   Bound    pvc-3602d021-da6d-466e-8440-c79653339ee5   2Gi        RWX            nfs-client     14s
```

Here is an example pod that will write data to our new PVC.
**nfs-test-pod.yml**

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-using-nfs
spec:
  volumes:
    - name: nfs-volume
      persistentVolumeClaim:
        claimName: qspvc
  containers:
    - name: apptest
      image: alpine
      volumeMounts:
        - name: nfs-volume
          mountPath: /var/nfs
      # Write to a file inside our NFS
      command: ["/bin/sh"]
      args: ["-c", "while true; do date >> /var/nfs/dates.txt; sleep 5; done"]
```
**Create test pod**

```shell
$ kubectl create -f nfs-test-pod.yml
pod/pod-using-nfs created
```

**Read from storage**

```shell
$ kubectl exec -it pod/pod-using-nfs -- /bin/sh
/ # ls -l /var/nfs/dates.txt
-rw-rw-rw-    1 nobody   nobody         261 Jan 10 15:24 /var/nfs/dates.txt

/ # cat /var/nfs/dates.txt
Fri Jan 10 15:23:49 UTC 2020
Fri Jan 10 15:23:54 UTC 2020
Fri Jan 10 15:23:59 UTC 2020
Fri Jan 10 15:24:04 UTC 2020
Fri Jan 10 15:24:09 UTC 2020
Fri Jan 10 15:24:14 UTC 2020
Fri Jan 10 15:24:19 UTC 2020
Fri Jan 10 15:24:24 UTC 2020
Fri Jan 10 15:24:29 UTC 2020
Fri Jan 10 15:24:34 UTC 2020
```