# Configuring Environment

The examples in this IAM section use the IBM Cloud Platform Services Python SDK which can be installed using the command:

```shell
pip install --upgrade "ibm-vpc"
```

You will also need to export the following environment variables if they are not already set:

```shell
export VPC_REGION='REGION FOR YOUR VPC RESOURCES'
export IBMCLOUD_API_KEY='YOUR IBM CLOUD API KEY'
export RESOURCE_GROUP='NAME OF RESOURCE_GROUP TO USE FOR DEPLOYMENTS"
```

## Versioning

Calls to the VPC API require a major version as the first segment of the request path (`ex: /v1/`) and a date-based version as a query parameter in the format `version=YYYY-MM-DD`. For safety I set this to one day behind the current date using the `datetime` module:

```py
from datetime import datetime, timedelta

today = datetime.now()
date = today + timedelta(days = -1)
version_date = date.strftime("%Y-%m-%d")

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))

## Construct the VPC service and set the regional endpoint
vpcService = VpcV1(
    authenticator=authenticator,
    version=version_date,
    )

vpcServiceRegion = 'https://' + os.environ.get('VPC_REGION') + '.iaas.cloud.ibm.com/v1'

vpcService.set_service_url(vpcServiceRegion)
```

## Endpoints

VPC uses region specific API endpoints:

- US South (`us-south`): `https://us-south.iaas.cloud.ibm.com/v1`
- US East (`us-east`):  `https://us-east.iaas.cloud.ibm.com/v1`
- Toronto (`ca-tor`): `https://ca-tor.iaas.cloud.ibm.com/v1`
- United Kingdom (`eu-gb`): `https://eu-gb.iaas.cloud.ibm.com/v1`
- Germany (`eu-de`): `https://eu-de.iaas.cloud.ibm.com/v1`
- Tokyo (`jp-tok`): `https://jp-tok.iaas.cloud.ibm.com/v1`
- Osaka (`jp-osa`): `https://jp-osa.iaas.cloud.ibm.com/v1`
- Sydney (`au-syd`): `https://au-syd.iaas.cloud.ibm.com/v1`
- São Paulo (`br-sao`): `https://br-sao.iaas.cloud.ibm.com/v1` 
