# psmasawscmdlineinterfaceoper
##8. Scripting Tasks: Alternatives to Shell Scripts
###2 Demo: Using Lambda for SSH
select a lambda blueprint->lambda-canary  

####02:56
using python function:
```
import boto3
import logging
logger = logging.getLogger()
logger.setlevel(logging.INFO)

def trigger_handler(event,context):
  client = boto3.client('ec2')
  instDict = client.describe_instance(Filters=[{'Name':'tag:environment','Values':['Stage']}])
  hostList=[]
  for r in instDict['Reservations']:
    for inst in r['Instances']:
      hostList.append(inst['PrivateIpAccess'])
  client = boto3.client('lambda')
  logger.error("Getting ready")
  for host in hostList:
    
```


