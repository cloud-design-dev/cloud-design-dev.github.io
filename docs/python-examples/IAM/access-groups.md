# Access Group Examples

The examples in this document use the [IAM Access Groups][access-groups] Python SDK.

## Create New Access Group

The following example will create a new Access Group named `python-test-access-group`:

```python
import os
import json
from ibm_platform_services import IamAccessGroupsV2
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_cloud_sdk_core import ApiException

## Pull Account ID from Environment variable
account_id = os.environ.get('ACCOUNT_ID')

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))
accessGroupService = IamAccessGroupsV2(authenticator=authenticator)

def create_access_group(accessGroupService):
    print("Creating new access group:")
    acessGroup = accessGroupService.create_access_group(
        name='python-test-access-group',
        account_id=account_id,
        description='Example Access Group created using the Python SDK'
    ).get_result()

    newAccessGroup = acessGroup
    print(json.dumps(newAccessGroup, indent=2))

try:
  create_access_group(accessGroupService)
except ApiException as ae:
  print("Method failed")
  print(" - status code: " + str(ae.code))
  print(" - error message: " + ae.message)
  if ("reason" in ae.http_response.json()):
    print(" - reason: " + ae.http_response.json()["reason"])
```

### Example Output

```shell
Creating new access group:
{
  "id": "AccessGroupId-xxxxxxx-2d2f-4e30-xxxxxxx-bca01772f49a",
  "name": "cool-project-access-group",
  "description": "Example Access Group created using the Python SDK for our cool new project",
  "account_id": "xxxxxxx",
  "created_at": "2022-04-25T21:07:07Z",
  "created_by_id": "IBMid-xxxxxxx",
  "last_modified_at": "2022-04-25T21:07:07Z",
  "last_modified_by_id": "IBMid-xxxxxxx"
}
```

## List All Access Groups

List all of the Access Groups on the account:

```python
import os
import json
from ibm_platform_services import IamAccessGroupsV2
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_cloud_sdk_core import ApiException

## Pull Account ID from Environment variable
account_id = os.environ.get('ACCOUNT_ID')

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get(''))
accessGroupService = IamAccessGroupsV2(authenticator=authenticator)

def list_access_groups(accessGroupService):
    groupsList = accessGroupService.list_access_groups(
        account_id=account_id,
        limit=100
    ).get_result().get("groups")

    for group in groupsList:
        print(json.dumps(group, indent=2))
   
try:
  list_access_groups(accessGroupService)
except ApiException as ae:
  print("Method failed")
  print(" - status code: " + str(ae.code))
  print(" - error message: " + ae.message)
  if ("reason" in ae.http_response.json()):
    print(" - reason: " + ae.http_response.json()["reason"])
```

## Get Access Group

You will need to set the Access Group ID in the script. Replace `Your-Access-Group-ID-Here` with the actual Access Group ID.

```python
import os
import json
from ibm_platform_services import IamAccessGroupsV2
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_cloud_sdk_core import ApiException

## Pull Account ID from Environment variable
account_id = os.environ.get('ACCOUNT_ID')

# Set access group used in the get_access_group function
access_group_id = 'Your-Access-Group-ID-Here'

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))
accessGroupService = IamAccessGroupsV2(authenticator=authenticator)
     
def get_access_group(accessGroupService, access_group_id):
    accessGroup = accessGroupService.get_access_group(
        access_group_id
    ).get_result()

    ag = accessGroup
    print(json.dumps(ag, indent=2))
  
try:
  get_access_group(accessGroupService, access_group_id=access_group_id)
except ApiException as ae:
  print("Method failed")
  print(" - status code: " + str(ae.code))
  print(" - error message: " + ae.message)
  if ("reason" in ae.http_response.json()):
    print(" - reason: " + ae.http_response.json()["reason"])
```

### Example Output

```shell
{
  "id": "AccessGroupId-xxxxxxx-1471-440b-xxxxxxx-xxxxxxx",
  "name": "CDE VPC Infrastructure",
  "description": "Access to VPC related resources in the CDE Resource Group",
  "account_id": "xxxxxxx",
  "created_at": "2020-11-06T14:27:16Z",
  "created_by_id": "IBMid-xxxxxxx",
  "last_modified_at": "2021-07-23T16:54:39Z",
  "last_modified_by_id": "IBMid-xxxxxxx"
}

```

[access-groups]: https://cloud.ibm.com/apidocs/iam-access-groups?code=python