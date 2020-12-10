---
layout: default
title: IBM Cloud Rest API 
nav_order: 2
has_children: true
---

## Prerequisites
 - IBM Cloud [API Key](https://cloud.ibm.com/docs/account?topic=account-userapikey#manage-user-keys)
 - IAM Token generated from API Key
 - [jq](https://stedolan.github.io/jq/) installed (Not strictly required but useful for parsing API response) 

### Export IBM Cloud API Key
You can generate an API Key by navigating to the [API Keys](https://cloud.ibm.com/iam/apikeys) page in the IBM Cloud Portal and clicking **Create an IBM Cloud API Key**. Copy your newly created API Key and export it to your shell session. 

```shell
export IBMCLOUD_API_KEY=<Your IBM Cloud API Key>
```

With the API key exported we now need to generate an IAM token in order to interact with the various APIs. The following command will generate a new IAM token from your API Key and store it as the variable `iam_token`. 

```shell
iam_token=`curl -s -k -X POST -H "Content-Type: application/x-www-form-urlencoded" \
-H "Accept: application/json" --data-urlencode "grant_type=urn:ibm:params:oauth:grant-type:apikey" \
--data-urlencode "apikey=${IBMCLOUD_API_KEY}" "https://iam.cloud.ibm.com/identity/token" \ 
| jq -r '(.token_type + " " + .access_token)'`
```
