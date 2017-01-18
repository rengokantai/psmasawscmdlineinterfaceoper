# psmasawscmdlineinterfaceoper
##7. Complex Operations: Rotate and Expire AMI Backups
###2 Demo: Rotate and Expire AMI
create instance ami
```
#!/bin/bash

REGION=$1
AMI=$2
today=`/bin/date +%Y%m%d%H%M`

/usr/local/bin/aws --region $REGION ec2 create-image --instance-id $AMI --name $AMI\_$today
```











##8. Scripting Tasks: Alternatives to Shell Scripts
###2 Demo: Using Lambda for SSH
select a lambda blueprint->lambda-canary  

####02:56
using python function:
```
import boto3
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def trigger_handler(event, context):
    #Get IP addresses of EC2 instances
    client = boto3.client('ec2')
    instDict=client.describe_instances(
            Filters=[{'Name':'tag:environment','Values':['stage']}]
        )
    hostList=[]
    for r in instDict['Reservations']:
        for inst in r['Instances']:
            hostList.append(inst['PrivateIpAddress'])

    #Invoke worker function for each IP address
    client = boto3.client('lambda')
    logger.error("Getting ready to loop through host list")
    for host in hostList:
        print "Invoking worker_function on " + host
        logger.error("Invoking worker_function on " + host)
        invokeResponse=client.invoke(
            FunctionName='worker_function',
            InvocationType='Event',
            LogType='Tail',
            Payload='{"IP":"'+ host +'"}'
        )
        print invokeResponse

    return{
        'message' : "Trigger function finished plus info logs"
    }
```

script
```
sudo pip install virtualenv
mkdir ~/build
virtualenv /home/ec2-user/build
cd build
source ./bin/activate
sudo yum install gcc openssl-devel python-devel libffi-devel
pip install pycrypto
pip install paramiko
```

working function
```
import boto3
import paramiko
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)
def worker_handler(event, context):

    logger.error("Creating s3 client")
    s3_client = boto3.client('s3')
    #Download private key file from secure S3 bucket
    logger.error("Connecting to s3 for key file")
    s3_client.download_file('lambda-db-backups','keys/csmithaws.pem', '/tmp/csmithaws.pem')

    c = paramiko.SSHClient()
    c.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    host=event['IP']
    print "Connecting to " + host
    logger.error("Connecting to " + host)
    c.connect( hostname = host, username = "ec2-user", key_filename = "/tmp/csmithaws.pem" )
    print "Connected to " + host

    commands = [
	 "touch /tmp/hi_there`date`"
#        I can add other commands here which will all be executed on the remote hosts
        ]
    for command in commands:
        print "Executing {}".format(command)
        stdin , stdout, stderr = c.exec_command(command)
        print stdout.read()
        print stderr.read()

    return
    {
        'message' : "Script execution completed. See Cloudwatch logs for complete output" + host
    }
```
packaing script
```
cd ~/build/lib/python2.7/site-packages/
zip -r ~/worker_function.zip *
cd ../../../lib64/python2.7/site-packages/
zip -r ~/worker_function.zip *
cd ~
zip worker_function.zip worker_function.py
```
