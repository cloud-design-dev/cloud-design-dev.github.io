---
layout: default
title: Searching with the IBM Cloud CLI
parent: IBM Cloud CLI
nav_order: 1
---

# Command line examples for _ibmcloud resource search_

 - IaaS = Classic Infrastructure (SoftLayer) Resources.  
 - Cloud = IBM Cloud Resources.  


## Search Cloud Resources (Cloud)
```shell
$ ibmcloud resource search 'name:devcluster'
```

### Search by resource name and return CRN  (Cloud)
```
$ ibmcloud resource search 'name:devcluster' --output json | jq -r '.items[].crn'
```

## Search by resource tag (Cloud)
```shell
$ ibmcloud resource search 'tags:ryantiffany' --output json
```

### Return resource names (Cloud)
```shell
$ ibmcloud resource search 'tags:ryantiffany' --output json | jq -r '.items[].name'
```

### Return resource CRNs (Cloud)
```shell
$ ibmcloud resource search 'tags:ryantiffany' --output json | jq -r  '.items[].crn'
```

### Return resource types  (Cloud)
```shell
$ ibmcloud resource search 'tags:ryantiffany' --output json | jq -r  '.items[].type'
```

## Search classic infrastructure (IaaS)
```shell
$ ibmcloud resource search -p classic-infrastructure --output json
```

### Search classic infrastructure by tag (IaaS)
```shell
$ ibmcloud resource search "tagReferences.tag.name:ryantiffany" -p classic-infrastructure --output json
```

#### Return resource types (IaaS)
```shell
$ ibmcloud resource search "tagReferences.tag.name:ryantiffany" -p classic-infrastructure --output json | jq -r '.items[].resourceType'
```

#### Search by tag and filter on virtual instances (IaaS)
```shell
$ ibmcloud resource search "tagReferences.tag.name:ryantiffany _objectType:SoftLayer_Virtual_Guest" -p classic-infrastructure --output json 
```

#### Search IaaS Virtual instances by Tag and return FQDNs
```shell
$ ibmcloud resource search "tagReferences.tag.name:ryantiffany _objectType:SoftLayer_Virtual_Guest" -p classic-infrastructure --output json | jq -r '.items[].resource.fullyQualifiedDomainName'
```

#### Search IaaS Virtual instances by Tag and return instance ID's 
```shell
$ ibmcloud resource search "tagReferences.tag.name:<tag> _objectType:SoftLayer_Virtual_Guest" -p classic-infrastructure --output json | jq -r '.items[].resource.id'
