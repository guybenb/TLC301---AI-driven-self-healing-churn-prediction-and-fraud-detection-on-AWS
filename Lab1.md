# Lab 1
##  what are we trying to do ?

In this lab we are going to build, train and deploy a churn prediction model using [Amazon SageMaker ](https://aws.amazon.com/sagemaker/).

We will then use [AWS Lambda](https://aws.amazon.com/lambda/) the invoke the Churn prediction endpoint that we have deployed 

##  What can I actually do with that Churn prediction endpoint ?

Great question!

We will then look on one use case where we have an [Amazon Connect](https://aws.amazon.com/connect/) instance.

We will deploy a simple call flow, and use the lambda function that we created in order to trigger the endpoint we created.

The result 'Churn'/'no Churn', will be used to decide to which queue shall we route the customer to - Expert/standard Queue. 

##  Do I need to have any machine learning knowledge in order to complete this lab ?

Absolutely not. All you need to do is follow the lab guide, ask questions when you have doubt and enjoy the workshop! 


## Solution Architecture

In the below diagram you can see a demo that we have built for an 'Agent Dashboard' which is using different AWS 
Services to do real time transcription and sentiment analytics and then visualize that to an agent.

It will also route the call to a Expert/Default queue once there is an incoming call using a churn prediction endpoint. This is what we are going to implement in this lab.
 
![image](https://user-images.githubusercontent.com/39404214/69583427-c8861280-0fd2-11ea-9cab-b27f69829613.png)


## Build, train and deploy a churn prediction model using Amazon SageMaker

* login to AWS Console 
* in the search bar enter 'sagemaker' and select 'Amazon SageMaker'
* click on on 'notebook instances' and create a notebook instance
* enter a name for your notebook instance
* select `ml.m5.xlarge` as your notebook instance type 
* under 'IAM role' create a new role 
* select 'Any S3 bucket' and click create new role
![image](https://user-images.githubusercontent.com/39404214/68408100-aab85100-017c-11ea-927a-9b29a37dc495.png)

* go ahead and click on 'Create notebook instance'

![image](https://user-images.githubusercontent.com/39404214/68397601-5eb0e080-016b-11ea-864d-df6f376cfec7.png)
* under 'notebook instance' once your instance status will show 'InService' , click on 'OpenJupyter' under Actions
![image](https://user-images.githubusercontent.com/39404214/68397725-8bfd8e80-016b-11ea-88af-1f6a8e391bd1.png)
* click on the 'SageMaker Examples' tab
* click on 'Introduction to Applying Machine Learning'
* click on the'use' button next to `xgboost_customer_churn.ipynb`
![image](https://user-images.githubusercontent.com/39404214/68398082-17771f80-016c-11ea-95dc-186cbe95e8bd.png)
* select `Conda_python3` as your kernel
* go through the SageMaker notebook
* point to your S3 bucket where the sample dataset will be downloaded to.

![image](https://user-images.githubusercontent.com/39404214/68398323-7472d580-016c-11ea-8f08-2d0777698fc9.png)

* you can now click on the 'Cell' tab. Run upto Step 17

![image](https://user-images.githubusercontent.com/39404214/68398566-dfbca780-016c-11ea-997b-618a304a4a3d.png)

Go through the notebook and examine the different steps:

1. data exploration
1. model training
1. Host

Once the model will be deployed and an endpoint will be created , we can move to the next stage.



## Create a lambda which will invoke the SageMaker endpoint

* go to the lambda console
* create a new lambda function
* select 'author from scratch'
* give a name for your lambda function
* select python 3.7 runtime
* make sure your lambda function has permission to invoke sagemaker endpoint 
![image](https://user-images.githubusercontent.com/39404214/69729998-2fb0dd80-111f-11ea-86b3-1c673e48d3a3.png)
* click on 'create function'
* copy the following lambda function to the console: Makesure you put the end point generated in SageMaker

```python
import os
import io
import boto3
import json
import csv
# grab environment variables
#ENDPOINT_NAME = os.environ['ENDPOINT_NAME']
runtime= boto3.client('runtime.sagemaker')
def lambda_handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))
    data = json.loads(json.dumps(event))
    payload ="0,47,28,141.3,94,168.0,108,113.5,84,7.8,2,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,1,0,0"
    print(payload)
    response = runtime.invoke_endpoint(EndpointName='xgboost-2019-03-22-11-38-32-449',
                                       ContentType='text/csv',
                                       Body=payload)
```

* save 

The payload will contain the customer sample in a CSV format which we will 'inject' into the endpoint .

As you can see above we have hard-coded in this example the customer data which should invoke the endpoint returning 'Churn'





## Integrate your Churn Prediction endpoint with Amazon connect (optional)

We would like to create an amazon connect instance which will have 2 agent queues:

1. basic queue
1. expert queue

The logic is that whenever there is an incoming call in real time there is a lambda function which triggers the churn prediction endpoint based on the customer data and according to the result (churn/not churn) the call will be routed 
between the queues.

We would create a  [Connect contact flows](https://docs.aws.amazon.com/connect/latest/adminguide/connect-contact-flows.html).

After creating an amazon connect instance, you can then import a contact flow we have created before the workshop).
Once you have imported the contact flow, you can then point to your lambda function.
Next step will be to generate an incoming call and observe how the call is being routed to an expert queue.

![image](https://user-images.githubusercontent.com/39404214/67765067-2ad70c00-fa43-11e9-8918-477b83cd93c7.png)



