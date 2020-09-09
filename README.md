Detectify BugCrowd Integration 

The Detectify-BugCrowd integration triggers a Lambda function every time a Detectify web app vulnerability scan is finished and every new finding is then collected by the Lambda function and pushed to the according to Bug Bounty Program on BugCrowd side. With that automation, all newly detected vulnerabilities by Detectify will be treated as a duplicate on BugCrowd side.


Step 1:  Create a new user in AWS
This step is needed for deploying lambda on your Aws account. The option is located under Identity and Access Management (IAM). https://console.aws.amazon.com/iam/home?region=eu-west-1#/users
You need to add a new user and copy Access key ID and Secret key (save on local machine for feature use). 





Step 2: Serverless configuration 
If you don't have serverless installed in your computer, you can do it with the following command 
npm install -g serverless  https://www.serverless.com/framework/docs/providers/aws/guide/installation

After a successful installation with access and secret key, now we should set up a serverless framework.
serverless config credentials --provider aws --key key --secret secret

For more information visit serverless site: Serverless Framework Commands - AWS Lambda - Config Credentials
Step 3: Add secrets variables on AWS Secrets Manager
To avoid stealing sensitive data like secrets keys, tokens, env. variables we use AWS Secrets Manager.  
For lambda to work correctly, we should create four variables:
secrets/DETECTIFY_API_KEY
secrets/DETECTIFY_SECRET_KEY
secrets/BUG_CROWD_TOKEN
secrets/BUG_BOUNTY_UUID


You can add secrets via AWS CLI or with AWS Secrets Manager (https://eu-west-1.console.aws.amazon.com/secretsmanager/home?region=eu-west-1#/listSecrets ) 

If you chose via CLI, first make sure you have installed AWS CLI. Installing the AWS CLI version 2 - AWS Command Line Interface
I have used MacOS and command to installing AWS are: 
$ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
$ sudo installer -pkg AWSCLIV2.pkg -target /

For other systems find details in the documentation, the link is above. 


After successful installation you can run:
 aws secretsmanager create-secret --name secrets/BUG_BOUNTY_UUID --description "The Bounty uuid, represents the id for a project on which you are going to create submissions." --secret-string 357a81ca-92b7-4b60-9548-5a64533c5cca

And secret with name secrets/BUG_BOUNTY_UUID and with value 357a81ca-92b7-4b60-9548-5a64533c5cca is created 
 Tutorial: Creating and Retrieving a Secret - AWS Secrets Manager

How to get these secrets is explained in the following steps. 
Step 4: Detectify API and secret key 
First, navigate to your team profile:

Then go to API keys:

Here generate a new key with the following permissions set to true:

Step 4: BugCrowd Token
Navigating to Api credentials section from https://tracker.bugcrowd.com/ url.
 













You will be able to create an auth token which we use for authentication. After creating the auth token you only need is to copy and paste it to the AWS Secrets Manager. For more details visit https://docs.bugcrowd.com/reference#authentication. 

Step 5: Get BugCrowd bounty uuid
The Bounty uuid, represents the id for a project on which you are going to create submissions.
This uuid can be obtained e.g. via postman, https://api.bugcrowd.com/bounties url or in an easier way if you invoke your AWS url (see step 8 ) or run the program and on localhost:3000( make a sure you have created env file with all of these variables). There you can see all projects available for your token (keep in mind that you first enter bugCrowd auth token in AWS Secrets Manager). 

This is an example JSON from http:localhost:3000 
After getting the uuid you need to create variable secrets/BUG_BOUNTY_UUID with the proper value 

Step 7: Deploying to AWS 
After you successfully finished all these steps, you can deploy your lambda function.
serverless deploy

Serverless Framework Commands - AWS Lambda - Deploy
Step 8:  Obtain AWS endpoint 
After deploying, you will need to obtain the AWS endpoint because as this will be configured within Detectify as a Webhook address. This can be done through API Gateway. You need to navigate to https://eu-west-1.console.aws.amazon.com/apigateway/main/apis?region=eu-west-1
Find your lambda function and click on it. (you probably will have two functions for dev and prod stage) 


Go to stages section, select the particular stage and you will be able to see invoke url on the right top side like in the picture below. 

Step 9: Setup detectify webhook
Navigate to your scan profile and there within the settings to integrations: Here you can add a webhook like this:

Replace the URL with your AWS endpoint. IMPORTANT to set the new findings options to not generate duplicate entries.
