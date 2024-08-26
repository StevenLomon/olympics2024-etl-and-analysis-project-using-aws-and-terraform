This is a Data Engineering project that models the Spotify API project mentioned in this video: https://www.youtube.com/watch?v=X8Fin2zMKK4 by Darshil Parmar: https://www.linkedin.com/in/darshil-parmar/
Instead of getting data from the Spotify API, I will be getting data from the Paris Olympics 2024 API: https://data.paris2024.org/api/explore/v2.1/console
The AWS Services that will be used are S3, AWS Glue (with AWS GlueCrawler), Amazon CloudWatch, AWS Lambda and Amazon Athena.
Terraform will be used to automate and manage the provisioning of AWS resources involved in the data pipeline.

I started by authorizing and getting my hands on the data from the API  
While waiting for the request access, I started setting up an IAM User that will be used during the project. This user will only have full access to all of the services that are used in the project and these services only as per the Principle of Least Privilege. The user will both have console access (with MFA) and programmatic access via the CLI and the user will also be disabled by the end of the project to minimuze security risks.  
When applying policies, I stumbled upon the two types of CloudWatch for the first time: AWS CloudWatchEvidently and AWS CloudWatchRUM (Real User Monitoring). CloudWatchEvidently is moreso used for A/B testing and feature flag management and so for this project CloudWatchRUM will instead be used. This is also more in line with what I learned when studying for the Cloud Practitioner Exam haha  
My own personal note: to access the AWS CLI, it's not awscli or aws-cli, it's aws configure
Since all of the services that will be used throughout the project are avaialble in the Stockholm (eu-north-1) region and it's the closest one I'm located to, this is the one that will be used.  

I then created the provider.tf file. This is my first time working with Terraform so everything is new but exciting and I'm focusing on failing forward! The official Terraform documentation has a super handy button that says "Use this Provider" that allows us to quickly set up our provider: https://registry.terraform.io/providers/hashicorp/aws/latest/docs  
With our terraform version block and aws provider block with the region we want to use in provider.tf written, we can run terraform init in our terminal (with Terraform also installed which I did earlier following this: https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) to initalize our Terraform back-end. We are displayed with "Terraform has been successfully initialized!"  
Both the .terraform directory and the .terraform.lock.hcl file are both added to .gitignore since they're both automatically generated and managed by Terraform.  

main.tf and variables.tf were created to handle the provisioning of AWS resources. I starting by creating an S3 Bucket for the Data Lake.   
I'm also making sure to use the variable name in variables.tf rather than writing them in main.tf. Single source of truth.

I decided, since I am pressed for time for job interview deadlines, to finish this project without Terraform and "re-do" it with Terraform in retrospect

So next up is setting up boto3 and creating an S3 object  
Since I still didn't have access to the official API data, I downloaded some Olympics data from Kaggle. All of this data was store in a directory called 'data' in the main directory and was uploaded to S3 via os operations. New for me was os.walk and os.path.relpath  

I decided to switch my approach once again since I remembered that RapidAPI is a thing! So rather than getting official data from the official API, for now I will be getting my data from this unofficial API: https://rapidapi.com/belchiorarkad-FqvHs2EDOtP/api/olympic-sports-api  
Since RapidAPI uses an API key, I set up dotenv and included it in .gitignore  

At this point I set up the Lambda function to extract the data. This is my first time writing a Lambda function and it will also be my first time using AWS CloudWatch. As I was setting up the Lambda funciton I got this error:  
User: arn:aws:iam::058264546342:user/olympics-user1 is not authorized to perform: iam:CreateRole on resource: arn:aws:iam::058264546342:role/service-role/OlympicsDataExtraction-role-gxqt7ypj because no identity-based policy allows the iam:CreateRole action  
The workaround was to create a custom policy so that no more control than needed was added. Only creating an attaching roles (and others needed to make those work) was needed and so a new policy was created using the following JSON and attached to the IAM User:  
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PassRole",
                "iam:PutRolePolicy",
                "iam:GetRole"
            ],
            "Resource": "arn:aws:iam::058264546342:role/*"
        }
    ]
}
However, I was quickly faced with another error displaying that the IAM user needs to be able to create policies as well so the JSON was slightly modified to include this as well:  
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PassRole",
                "iam:PutRolePolicy",
                "iam:GetRole",
                "iam:CreatePolicy"
            ],
            "Resource": "*"
        }
    ]
}
With this, I was able to create the Lambda function!  



