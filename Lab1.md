###  what are we trying to do ?

in this lab we are going to build,train and deploy  a churn prediction model using [Amazon SageMaker ](https://aws.amazon.com/sagemaker/).

we will then use [AWS Lambda](https://aws.amazon.com/lambda/) the invoke the Churn prediction endpoint that we had deployed 

###  what can i actually do with that Churn prediction endpoint ?

Great question!

we will then look on one use case where we have  an [Amazon Connect ](https://aws.amazon.com/connect/) instance.

we will deploy a simple call flow , and use the lambda function that we created in order to trigger the endpoint we created.

the result 'Churn' /'no Churn' , will be used to decide to which queue shall we route the customer to - Expert/standard Queue. 

###  do i need to have any machine learning knowledge in order to complete this lab ?

absolutely no, all you need to do is follow the lab guide ,ask questions when you have doubt and enjoy the workshop! 


## solution architecture:

in the below diagram you can see a demo that we have built for an 'Agent Dashboard' which is using different AWS 
Services to do real time transcription and sentiment analytics and then visualize that to an agent .

it will also route the call to a Expert/Default queue once there is an incoming call using a churn prediction endpoint,and this is what we are going to implement in this lab.
 



![image](https://user-images.githubusercontent.com/39404214/69583427-c8861280-0fd2-11ea-9cab-b27f69829613.png)

 


### build,train and deploy a churn prediction model using Amazon SageMaker

* login to AWS Console 
* in the search bar enter 'sagemaker' and select 'Amazon SageMaker'
* click on on 'notebook instances' and create a notebook instance
* enter a name for your notebook instance
* select ml.m5.xlarge as your notebook instance type 
* under 'IAM role' create a new role 
* select 'Any S3 bucket' and click create new role
![image](https://user-images.githubusercontent.com/39404214/68408100-aab85100-017c-11ea-927a-9b29a37dc495.png)

* go ahead and click on 'Create notebook instance'

![image](https://user-images.githubusercontent.com/39404214/68397601-5eb0e080-016b-11ea-864d-df6f376cfec7.png)
* under 'notebook instance' once your instance status will show 'InService' , click on 'OpenJupyter' under Actions
![image](https://user-images.githubusercontent.com/39404214/68397725-8bfd8e80-016b-11ea-88af-1f6a8e391bd1.png)
* click on the 'SageMaker Examples' tab
* click on 'Introduction to Applying Machine Learning'
* click on the'use' button next to 'xgboost_customer_churn.ipynb'
![image](https://user-images.githubusercontent.com/39404214/68398082-17771f80-016c-11ea-95dc-186cbe95e8bd.png)
* select Conda_python3 as your kernel
* go through the sagemaker notebook
* point to your S3 bucket where the sample dataset will be downloaded to .

![image](https://user-images.githubusercontent.com/39404214/68398323-7472d580-016c-11ea-8f08-2d0777698fc9.png)

*you can now click on the 'Cell' tab and then select 'Run All' :

![image](https://user-images.githubusercontent.com/39404214/68398566-dfbca780-016c-11ea-997b-618a304a4a3d.png)



go through the notebook and examine the different steps:

1. data exploration
1. model training
1. Host

once the model will be deployed and an endpoint will be created , we can move to the next stage.



### create a lambda which will invoke the sagemaker endpoint

* go to the lambda console
* create a new lambda function
* select 'author from scratch'
* give a name for your lambda function
* select python 3.7 runtime
* make sure your lambda function has permission to invoke sagemaker endpoint 
![image](https://user-images.githubusercontent.com/39404214/69729998-2fb0dd80-111f-11ea-86b3-1c673e48d3a3.png)
* click on 'create function'
* copy the following lambda function to the console:

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

the payload will contain the customer sample in a CSV format which we will 'inject' into the endpoint .

as you can see above we have hard-coded in this example the customer data which should invoke the endpoint returning 'Churn'





### integrate your Churn Prediction endpoint with Amazon connect ( optional)

we would like to create an amazon connect instance which will have 2 agent queues:

1. basic queue
1. expert queue

the logic is that whenever there is an incoming call in real time there is a lambda function which triggers the churn prediction endpoint based on the customer data and according to the result (churn/not churn) the call will be routed 
between the queues .

we would create a  [Connect contact flows](https://docs.aws.amazon.com/connect/latest/adminguide/connect-contact-flows.html).

after creating an amazon connect instance , you can then import a contact flow we have created before the workshop).
once you have imported the contact flow , you can then point to your lambda function .
next step will be to generate an incoming call and observe how the call is being routed to an expert queue.

![image](https://user-images.githubusercontent.com/39404214/67765067-2ad70c00-fa43-11e9-8918-477b83cd93c7.png)



