# Single Zone VPC 

The following example will deploy a new VPC as well as a Public Gateway and Subnet in a single zone within the region.  

```py
import os
import json
from pprint import pprint
from ibm_vpc import VpcV1
from ibm_platform_services import ResourceManagerV2
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_cloud_sdk_core import ApiException
from haikunator import Haikunator
from datetime import datetime, timedelta

today = datetime.now()
date = today + timedelta(days = -1)
version_date = date.strftime("%Y-%m-%d")

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))

resourceService = ResourceManagerV2(authenticator=authenticator)

resource_group = (os.environ.get('RESOURCE_GROUP'))
resource_group_list = resourceService.list_resource_groups(
  account_id=(os.environ.get('IBM_ACCOUNT')),
  include_deleted=False,
).get_result()

rglist = resource_group_list['resources']
rg_id = rglist['name' == resource_group]['id']

## Construct the VPC service and set the regional endpoint 
vpcService = VpcV1(authenticator=authenticator)
vpcServiceRegion = 'https://' + os.environ.get('VPC_REGION') + '.iaas.cloud.ibm.com/v1'
vpcService.set_service_url(vpcServiceRegion)

## Pull zones based on region. Set deployment zone to first in region by default. 
## Todo: based on length of zones returned, create that number of pubgws and subnets
zones = vpcService.list_region_zones(os.environ.get('VPC_REGION')).get_result()['zones']
deployment_zone = zones[0]['name']

## Use Haikunator to generate a unique heroku like base name for resources.
## Handy while testing 
haikunator = Haikunator()
basename = haikunator.haikunate(token_length=0, delimiter='')

## Set up some parameters that will be used by multiple functions 
resource_group_identity_model = {}
resource_group_identity_model['id'] = rg_id
resource_group_id = resource_group_identity_model

zone_identity_model = {}
zone_identity_model['name'] = deployment_zone
zone = zone_identity_model

def create_vpc(vpcService):

    address_prefix_management = 'auto'
    classic_access = False
    name = (basename + "-vpc")
    
    new_vpc = vpcService.create_vpc(
        classic_access=classic_access,
        address_prefix_management=address_prefix_management,
        name=name,
        resource_group=resource_group_id
    ).get_result()

    print("\nCreating IBM Cloud VPC in " + os.environ.get('VPC_REGION') + " ----\n")
    newVpc = create_vpc(vpcService)
    print("Creation Complete. VPC Info: ----\nName: " + newVpc['name'] + "\nID: " + newVpc['id'] + "\nCRN: " + newVpc['crn'] + " ----\n----\n")
    print(newVpc['default_network_acl']['id'])

    return new_vpc

def create_pubgw(vpcService):
    vpc_identity_model = {'id': newVpc['id']}
    vpc = vpc_identity_model
    zone = zone_identity_model
    name = (basename + "-" + deployment_zone + "-gw")

    new_pubgw = vpcService.create_public_gateway(
        vpc,
        zone,
        name=name,
        floating_ip={},
        resource_group=resource_group_id
    ).get_result()

    newPubGw = create_pubgw(vpcService)
    print("Creation Complete. Pubgw Info: ----\nName: " + newPubGw['name'] + "\nID: " + newPubGw['id'] + "\nCRN: " + newPubGw['crn'])

    return new_pubgw

def create_subnet(vpcService):
    vpc_identity_model = {'id': newVpc['id']}
    vpc = vpc_identity_model
    zone = zone_identity_model
    name = (basename + "-" + deployment_zone + "-subnet")
    network_acl_identity_model = {}
    network_acl_identity_model['id'] = newVpc['default_network_acl']['id']

    public_gateway_identity_model = {}
    public_gateway_identity_model['id'] = newPubGw['id']

    subnet_prototype_model = {}
    subnet_prototype_model['ip_version'] = 'ipv4'
    subnet_prototype_model['name'] = name
    subnet_prototype_model['network_acl'] = network_acl_identity_model
    subnet_prototype_model['public_gateway'] = public_gateway_identity_model
    subnet_prototype_model['resource_group'] = resource_group_identity_model
    subnet_prototype_model['vpc'] =  vpc
    subnet_prototype_model['total_ipv4_address_count'] = 256
    subnet_prototype_model['zone'] = zone

    subnet_prototype = subnet_prototype_model
    new_subnet = vpcService.create_subnet(subnet_prototype).get_result()
    
    print("\nCreating VPC Subnet in " + deployment_zone +" ----\n")
    newSubnet = create_subnet(vpcService)
    print("Creation Complete. Subnet Info: ----\nName: " + newSubnet['name'] + "\nID: " + newSubnet['id'] + "\nCRN: " + newSubnet['crn'])
    return new_subnet

try:
  create_vpc(vpcService)
  create_pubgw(vpcService)
  create_subnet(vpcService)
except ApiException as ae:
  print("Method failed")
  print(" - status code: " + str(ae.code))
  print(" - error message: " + ae.message)
  if ("reason" in ae.http_response.json()):
    print(" - reason: " + ae.http_response.json()["reason"])
```