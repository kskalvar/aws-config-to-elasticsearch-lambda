AWS Config Snapshots to AWS Elasticsearch using Lambda
======================================================

This solution shows how to automate the ingestion of your AWS Config snapshots into the ElasticSearch/Logstash/Kibana (ELK) stack for searching and mapping your AWS environments.  We'll be using a Lambda function which will automatically be fired when a new file is upload to the S3 bucket AWS Config is using to store Snapshot files.  This readme updates an article "How to Analyze AWS Config Snapshots with ElasticSearch and Kibana by Vladimir Budilov" referenced below and provides a more basic step by step process.

## Configure AWS Config Service

Use the AWS Console to configure the AWS Config Service.  This is a step by step process.

### AWS Config Console
Get started

#### Settings
Resource types to record  
```
All resources: checked
```  

Amazon S3 bucket  
```
Create a bucket: selected
```

AWS Config role  
```
Create a role: selected
```  
Next

#### AWS Config rules
Next

#### Review
Confirm  

### AWS Resources Created
S3 Bucket  
```
config-bucket-<id>
```   
IAM Roles  
```
config-role-us-east-1
```    
IAM Policies  
```
Customer managed:  config-role-us-east-1_AWSConfigDeliveryPermissions_us-east-1
```  

## Configure AWS Elasticsearch Service

Use the AWS Console to configure the AWS Elasticsearch Service. This is a step by step process.

### AWS Elasticsearch Console
Create a new domain

#### Define domain
Elasticsearch domain name:  
```
aws-config-es
```    
Elasticsearch version:  
```
5.5
```  
Next

#### Configure cluster
Instance count:  
```
1
```  
Instance type:  
```
t2.small.elasticsearch
```  
Next

#### Set up access
Network configuration  
```
Public access
```  
Access policy  
Select a template pull-down  
```
Allow open access to the domain
Note:  acknowledge risk pop-up
```  
Next

#### Review
Confirm

#### AWS Elasticsearch Resources Created

From the AWS Elasticsearch Console you will see ```aws-config-es``` domain configuration status.  "Configuration state” shows the status of the just created domain and initially will display “Loading”.  Once it shows “Active” the domain will be ready to access.  Node configuration took 12 minutes to become active for this example.  

Drill down to the aws-config-es domain and select the "Overview" tab.  

Of specific interest for this task are:  

```
"Endpoint: <Endpoint URL>"
"Kibana: <Kibana URL>"
```
## Configure AWS Lambda Function

Use the AWS Console to configure the AWS Lambda function. The Lambda function will be used to import AWS Config Snapshots into Elasticsearch automatically.  This is a step by step process.

### AWS Lambda Console
Create a function

#### Create function
```
Author from scratch
	Name: aws-config-lambda
	Runtime:  Python 2.7
	Role: Create new roll from template(s)
	Role name: aws-config-lambda-role
	Policy templates: S3 object read-only permissions
```  
Create function

#### aws-config-lambda (Configuration tab)

Designer section "Add triggers from the list on the left"
```
Add triggers
Select "S3"
```

Configure triggers Section
Event type:  PUT
Enable trigger: checked
Add
Save


Designer section
Highlight "amp-sight-lambda" 

Function code Section
Code entry type pull-down
```
Upload a .ZIP file
```
Upload

Update lambda_funtion with ```<Endpoint URL>``` from AWS Elasticsearch Resources Created
```
destination=<Endpoint URL>
```
Save

Basic settings Section
Timeout: 3 min 0 sec
Save

## Configure Kibana
Use the Kabana Console to configure the index pattern and time filter.  This is a step by step process.

### Access Kabana via Web Browser 
```
Using <Kibana URL> from "AWS Elasticsearch Resources Created"
```

#### Management View
Index name or pattern
```
*
```
Time Filter field name
```
snapshotTimeIso
```
Create

##### Discover View
Go to the Discover View to browse data


## References
How to Analyze AWS Config Snapshots with Elasticsearch and Kibana  
https://aws.amazon.com/blogs/developer/how-to-analyze-aws-config-snapshots-with-elasticsearch-and-kibana/

Import your AWS Config Snapshots into Elasticsearch  
https://github.com/awslabs/aws-config-to-elasticsearch

Understanding the basic components of AWS Config will help you get the most out of this service  
https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html

Creating and Configuring Amazon Elasticsearch Service Domains  
https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html

Starting from Elasticsearch 6.0, all REST requests that include a body must also provide the correct content-type for that body  
https://www.elastic.co/blog/strict-content-type-checking-for-elasticsearch-rest-requests

