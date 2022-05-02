# Getting Started

The examples in this document use various IBM Cloud SDKs to interact with the various cloud services. You can install the modules as needed for each example, or you can install them all at once using `pip`:

```shell
pip install --upgrade ibm-platform-services
pip install --upgrade ibm-cloud-networking-services
pip install --upgrade ibm-cloud-databases
pip install --upgrade ibm-cloud-sdk-core
pip install --upgrade ibm-schematics
pip install --upgrade ibm-vpc
```

## Authentication

IBM Cloud services support a variety of authentication options but for these guides focus on the token-based Identity and Access Management ([IAM][iam]) authentication. With IAM authentication, you supply an API key and the authenticator will exchange that API key for an access token (a bearer token) by interacting with the IAM token service. The access token is then added (via the Authorization header) to each outbound request to provide the required authentication information.

Access tokens are valid only for a limited amount of time and must be periodically refreshed. The IAM authenticator will automatically detect the need to refresh the access token and will interact with the IAM token service as needed to obtain a fresh access token, relieving the SDK user from that burden.

For instance if we were interacting with the IAM Access Group service our authentication and client set up would look like this:

```python
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator

authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))
accessGroupService = IamAccessGroupsV2(authenticator=authenticator)
```

For more details about the construction of the IAM authentication flow see [IBM Cloud SDK Common][ibm-sdk] documentation.

[iam]: https://cloud.ibm.com/docs/account?topic=account-iamoverview
[ibm-sdk]: https://github.com/IBM/python-sdk-core/blob/main/Authentication.md#identity-and-access-management-authentication-iam