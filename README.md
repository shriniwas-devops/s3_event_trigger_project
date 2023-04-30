# AWS S3 Event Triggering

**NOTE**: REPLACE YOUR AWS ACCOUNT ID IN THE LAMBDA FUNCTION CODE.

AWS S3 Event triggering is a very popular project used by top companies in the Industry.

Here are some examples of top companies that use S3 event triggering:

**Netflix**: Netflix use S3 event triggering to automatically process video files uploaded to Amazon S3, enabling seamless content ingestion and processing.

**Airbnb**: This lodging and homestays aggregator use S3 event triggering to automatically process and analyze data stored in Amazon S3, such as guest reviews and booking information.

**Expedia**: They use S3 event triggering to automatically process and analyze data stored in Amazon S3, such as travel bookings, user profiles, and pricing information, to power their personalized travel recommendations and search features.


![Screenshot 2023-04-14 at 7 06 46 PM](https://user-images.githubusercontent.com/43399466/232058778-a7299e9b-9892-471c-a05d-14d773b5b333.png)



The scenario here — What all services are used here(Let’s take a sample example of Netflix here)
AWS S3 Event triggering is a very popular shell scripting used by several companies in the Industry.

The Amazon S3 notification feature enables you to receive notifications when a certain event occurs inside your S3 Storage bucket. To get notifications, first, add a notification configuration that reads the event you want Amazon S3 Bucket to publish and the destination where Amazon S3 will send the notifications. This configuration is stored in the notification sub-resource that is associated with a bucket.

Each time a new video/series/documentary is uploaded in S3 Storage Bucket, you receive a notification. Most organizations use S3 since it is cost-efficient.
Under the hood, the S3 bucket triggers Lambda Function. Since Lambda is a server less compute, it allows you to run code that can be written in any programming language. Also, it follows Pay As You Go pricing where you are charged based on the usage.
This information is sent to SNS (Simple Notification Service), which is a fully managed messaging service provided by Amazon Web Services (AWS). SNS provides reliable message delivery, scalability, and fault tolerance, making it suitable for building robust and scalable messaging systems in the cloud.
From SNS, it depends from platform to platform. If it is a simple blog posting website, the content can be stored in a S3 bucket and email notifications can be sent to all the subscribers. Whereas in case of Netflix, there would be some additional queuing, transcoding the video to different formats, extracting metadata, or performing other media processing tasks.
IAM — IAM stands for Identity and Access Management. IAM allows you to create and manage roles, which are sets of permissions that can be used to access AWS resources, such as EC2 instances, Lambda functions, or other services. In this specific scenario, the Lambda function needs to have permission to be invoked by S3 Bucket and invoke SNS. Hence we will need to grant IAM permission to Lambda Function.



Types of Event Notifications:-
New object created events — Amazon S3 sends a notification when an object is created.
Object removal events — Amazon S3 sends a notification upon deletion of an object. It supports two delete options. One is Permanently Delete and the other is Delete Marker Created.
Restore object events — Amazon S3 allows restoration of objects archived to the S3 Glacier storage classes.
Reduced Redundancy Storage (RRS) object lost events — Amazon S3 notifies by delivering a message when it detects that an object of the RRS storage class has been lost.
Supported Destinations:
Amazon Simple Notification Service (Amazon SNS) topic — Amazon SNS is a fully managed, flexible push messaging service.
Amazon Simple Queue Service (Amazon SQS) queue — Amazon SQS is a scalable and fully managed message queuing service. SQS can be used to transmit any volume of data without requiring other services to be always available.
AWS Lambda — AWS Lambda is a server less compute service that makes it easy for you to build applications that respond quickly to new information. AWS Lambda runs written code in response to events such as image uploads, in-app activity, website clicks, or outputs from connected devices.
Here are the steps we will need to follow:-

Step 1 — Launch an AWS EC2 T2 Micro Instance

Step 2 — Configure AWS CLI

Step 3 —Clone the Repository

Step 4— Some Important Commands to enter to avoid error

Step 5— Execute the shell script and check for email confirmation

Now, lets dig deeper into each of these steps:-

Step 1 — Launch an AWS EC2 Instance. Enable HTTP and HTTPS traffic. You can create a new key pair or use an existing one.


![image](https://user-images.githubusercontent.com/122585172/235352002-fbbb5797-8f36-4e7e-a0c7-78bec142d161.png)


Once the instance is in a running state, you can connect it via console or using the SSH Key Pair.

Step 2— Configure AWS CLI.

Once you have connected to your EC2 Instance, you need to install and configure AWS CLI using the below command

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
sudo apt install unzip
unzip awscliv2.zip 
sudo ./aws/install
aws configure

On the top right corner of your AWS Console, click on Security Credentials

To configure the AWS CLI, you can follow these steps:

Install the AWS CLI: If you haven’t already installed the AWS CLI on your local machine, you can download and install it from the official AWS CLI documentation: https://aws.amazon.com/cli/.
Open a command prompt or terminal window on your local machine.
Run the “aws configure” command: Type “aws configure” in the command prompt or terminal window and press Enter. This will start the configuration process.
Provide AWS access key ID: Enter your AWS access key ID, which is a unique identifier for your AWS account. Press Enter.
Provide AWS secret access key: Enter your AWS secret access key, which is a secret key associated with your AWS access key ID. Press Enter.
Provide default region name: Enter the default AWS region name that you want to use for your AWS CLI commands. This is the region where your resources will be created by default. Press Enter.
Provide default output format: Choose the default output format for your AWS CLI commands. You can choose either “json”, “text”, “table”, or “yaml”. Press Enter.


Step 3— You can fork my repository and then using the clone command, you can get the repository from Github to your EC2 Instance.



git clone https://github.com/writetoritika/shell-scripting-projects.git
cd shell-scripting-projects/
cd aws-event-triggering/


Step 4 — Some important command that you should enter before you execute the shell script

sudo apt-get install jq
sudo apt-get install zip
sudo apt-get update

Step 5— Using Shell Script, we will create S3 Bucket, Lambda Function, and Assign IAM Role(To be invoked by S3 Bucket and invoke SNS).

Go to your AWS Terminal

vi s3-notification-triggers.sh

These are the contents of the script

#!/bin/bash

set -x

# Store the AWS account ID in a variable
aws_account_id=$(aws sts get-caller-identity --query 'Account' --output text)

# Print the AWS account ID from the variable
echo "AWS Account ID: $aws_account_id"

# Set AWS region and bucket name
aws_region="us-east-1"
bucket_name="ritika-ultimate-bucket"
lambda_func_name="s3-lambda-function"
role_name="s3-lambda-sns"
email_address="writetoritika@gmail.com"

# Create IAM Role for the project
role_response=$(aws iam create-role --role-name s3-lambda-sns --assume-role-policy-document '{
  "Version": "2012-10-17",
  "Statement": [{
    "Action": "sts:AssumeRole",
    "Effect": "Allow",
    "Principal": {
      "Service": [
         "lambda.amazonaws.com",
         "s3.amazonaws.com",
         "sns.amazonaws.com"
      ]
    }
  }]
}')

# Extract the role ARN from the JSON response and store it in a variable
role_arn=$(echo "$role_response" | jq -r '.Role.Arn')

# Print the role ARN
echo "Role ARN: $role_arn"

# Attach Permissions to the Role
aws iam attach-role-policy --role-name $role_name --policy-arn arn:aws:iam::aws:policy/AWSLambda_FullAccess
aws iam attach-role-policy --role-name $role_name --policy-arn arn:aws:iam::aws:policy/AmazonSNSFullAccess

# Create the S3 bucket and capture the output in a variable
bucket_output=$(aws s3api create-bucket --bucket "$bucket_name" --region "$aws_region")

# Print the output from the variable
echo "Bucket creation output: $bucket_output"

# Upload a file to the bucket
aws s3 cp ./example_file.txt s3://"$bucket_name"/example_file.txt

# Create a Zip file to upload Lambda Function
zip -r s3-lambda-function.zip ./s3-lambda-function

sleep 5
# Create a Lambda function
aws lambda create-function \
  --region "$aws_region" \
  --function-name $lambda_func_name \
  --runtime "python3.8" \
  --handler "s3-lambda-function/s3-lambda-function.lambda_handler" \
  --memory-size 128 \
  --timeout 30 \
  --role "arn:aws:iam::956919395764:role/$role_name" \
  --zip-file "fileb://./s3-lambda-function.zip"

# Add Permissions to S3 Bucket to invoke Lambda
aws lambda add-permission \
  --function-name "$lambda_func_name" \
  --statement-id "s3-lambda-sns" \
  --action "lambda:InvokeFunction" \
  --principal s3.amazonaws.com \
  --source-arn "arn:aws:s3:::$bucket_name"

# Create an S3 event trigger for the Lambda function
aws s3api put-bucket-notification-configuration \
  --region "$aws_region" \
  --bucket "$bucket_name" \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
        "LambdaFunctionArn": "arn:aws:lambda:us-east-1:956919395764:function:s3-lambda-function",
        "Events": ["s3:ObjectCreated:*"]
    }]
}'

# Create an SNS topic and save the topic ARN to a variable
topic_arn=$(aws sns create-topic --name s3-lambda-sns --output json | jq -r '.TopicArn')

# Print the TopicArn
echo "SNS Topic ARN: $topic_arn"

# Trigger SNS Topic using Lambda Function


# Add SNS publish permission to the Lambda Function
aws sns subscribe \
  --topic-arn "$topic_arn" \
  --protocol email \
  --notification-endpoint "$email_address"

# Publish SNS
aws sns publish \
  --topic-arn "$topic_arn" \
  --subject "A new object created in s3 bucket" 
  
  To execute this shell script,

./s3-notification-triggers.shSo, what we have done via Shell Script is that we have

. Created IAM Role

. Created S3 Bucket

. Created Lambda Function

. Created SNS Topic and added personal email as a subscription to this SNS Topic.

Once your script is executed, you will receive an email asking about your confirmation of the subscription. If you go ahead and confirm, you will receive a notification email if anything new is added to the S3 Bucket.

Now, when you will go to your AWS Console, you will see the following resources created

S3 Bucket


![image](https://user-images.githubusercontent.com/122585172/235352152-2097d64c-9db0-47cb-adfb-39cc69a13966.png)


Lambda Function

![image](https://user-images.githubusercontent.com/122585172/235352156-7f8845ed-f714-49bb-a559-fa2bb776f6d4.png)



SNS Topic
![image](https://user-images.githubusercontent.com/122585172/235352172-a87e6de1-c5a2-4cae-b626-024002736e2b.png)


New IAM Role(S3-Lambda-SNS) — SNS Full Access and Lambda Full Access


![image](https://user-images.githubusercontent.com/122585172/235352178-233d7775-e8f4-416c-8b64-bf2fecb145c0.png)



Check your email for confirmation regarding subscription


![image](https://user-images.githubusercontent.com/122585172/235352183-b974b3df-0fed-4b2d-8be0-f21e3cc2b00c.png)



