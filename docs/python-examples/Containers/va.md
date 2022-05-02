# Vulnerability Advisor Examples

## List the Vulnerability Advisor status for all container images

```python
import os
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_cloud_sdk_core import ApiException
from ibm_container_registry.vulnerability_advisor_v3 import *

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))

account = os.environ.get('ACCOUNT_ID')
accept_language = 'en_US'

vulnerabilityAdvisorService = VulnerabilityAdvisorV3(
    authenticator=authenticator,
    account=account
    )

def get_all_scan_results(vulnerabilityAdvisorService):
  scanreport = vulnerabilityAdvisorService.account_status_query_path().get_result()
  image_report = scanreport['images']
  for image in image_report:
      print("Container Image: " + image['name'] + " was marked as " + image['status'] + " in the most recent VA scan.")

try:
  get_all_scan_results(vulnerabilityAdvisorService)
except ApiException as ae:
  print("Method failed")
  print(" - status code: " + str(ae.code))
  print(" - error message: " + ae.message)
  if ("reason" in ae.http_response.json()):
    print(" - reason: " + ae.http_response.json()["reason"])
```

### Example Output

```shell
Image: us.icr.io/rtiffany/nodejscloudantyzruj20211019:1-master-d45c0dba-20211019112141
VA Status: FAIL
Last Scan: 2022-04-22 18:27:05

Image: us.icr.io/rtiffany/rclone-sync:8
VA Status: FAIL
Last Scan: 2022-04-24 23:04:13

Image: us.icr.io/rtiffany/pythonbase-invoice:1
VA Status: FAIL
Last Scan: 2022-04-24 03:05:57

Image: us.icr.io/rtiffany/cde-mkdocs:1
VA Status: FAIL
Last Scan: 2022-04-22 16:44:34
```

## Get latest scan results for a container image

```python

import os
import json
from pprint import pprint
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_cloud_sdk_core import ApiException
from ibm_container_registry.vulnerability_advisor_v3 import *

## Construct IAM Authentication using IBMCLOUD_API_KEY Environment variable
authenticator = IAMAuthenticator(os.environ.get('IBMCLOUD_API_KEY'))
imageName =  "icr.io/YOUR-NAMESPACE/IMAGE_NAME:TAG"

account = os.environ.get('ACCOUNT_ID')
accept_language = 'en_US'

vulnerabilityAdvisorService = VulnerabilityAdvisorV3(
    authenticator=authenticator,
    account=account
    )

def va_scan_report(vulnerabilityAdvisorService, imageName):
  scan_report = vulnerabilityAdvisorService.image_report_query_path(
    name=imageName
  ).get_result()

  print(json.dumps(scan_report, indent=2))

try:
  va_scan_report(vulnerabilityAdvisorService, imageName=imageName)
except ApiException as ae:
  print("Method failed")
  print(" - status code: " + str(ae.code))
  print(" - error message: " + ae.message)
  if ("reason" in ae.http_response.json()):
    print(" - reason: " + ae.http_response.json()["reason"])
```