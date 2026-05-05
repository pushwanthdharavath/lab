lambda
------------------------------------
Basic python program---sample

🔹Step 1: aws – services-compute - lamdba
 Step 2: Create a New Lambda Function
1.Click Create function
2.Choose Author from scratch
3.Fill the details:
oFunction name: basic-python-lambda
oRuntime: Python 3.9 (or latest available)
oArchitecture: x86_64
4.Under change the default rule:
oSelect-use the existing rule- Lab Role
5.Click Create function

🔹 Step 3: Understand the Default Lambda Code
AWS provides a default Python template:
import json
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
Explanation:
Component Meaning
event Input data passed to Lambda
context Runtime info (memory, request ID, etc.)
lambda_handler Entry point function
statusCode HTTP response code
body Output response

🔹 Step 4: Write a Simple Python Lambda Program
Example: Add Two Numbers
import json
def lambda_handler(event, context):
    a = event.get('num1', 0)
    b = event.get('num2', 0)
    result = a + b
    return {
        'statusCode': 200,
        'body': json.dumps({
            'sum': result
        })
    }

🔹 Step 5: Deploy the Function
1.Click Deploy
2.Wait for the Successfully deployed message

🔹 Step 6: Create a Test Event
1.Click Test
2.Choose Create new event
3.Event name: testInput
4.Use this JSON input:
{
  "num1": 10,
  "num2": 20
}
5.Click Test

🔹 Step 7: View Output
You will see output like:
{
  "statusCode": 200,
  "body": "{\"sum\": 30}"
}
✔️ This means your Lambda executed successfully.

🔹 Step 8: Set General Configuration
1.Click General configuration → Edit
2.Set the following values:
Setting Value
Memory 128 MB
Timeout 3 seconds
Architecture x86_64
3.Click Save
✔ These values are sufficient for simple arithmetic operations.
-------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------

Elastic load balancer
WEEK-9- Implementation of Elastic Load Balancer with Auto Scaling using AWS EC2 Instances.
Elastic Load Balancer (ALB)
Step 1: Launch Two EC2 Instances
1.1 Launch EC2 Instance #1
1.Go to AWS EC2 Console → Launch Instance.
2.Name: webserver-1
3.Choose an AMI (Amazon Linux 2).
4.Select Instance Type (e.g., t2.micro).
5.Add Storage (default is fine).
6.Configure Security Group:
oAllow SSH (22) from your IP
oAllow HTTP (80) from anywhere (0.0.0.0/0)
7.Launch → Select/Create Key Pair → Launch.
Run these commands after connecting VM
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo  "This is Server 1" > /var/www/html/index.html

✅ Now you create second EC2 instances running Apache, with message This is Servier 2 each serving a simple web page.
Repeat it for second instance
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo  "This is Server 2" > /var/www/html/index.html


Step 2: Create a Load Balancer
AWS provides Elastic Load Balancer (ELB). We will create an Application Load Balancer (ALB).

2.1 Configure Security Group for Load Balancer
Go to EC2 → Security Groups → Create Security Group
oName: lb-sg
oAllow HTTP (80) from anywhere
oOptional: HTTPS (443) if using SSL

2.2 Create the Load Balancer
1.Go to EC2 → Load Balancers → Create Load Balancer → Application Load Balancer
2.Basic Configuration:
oName: my-alb
oScheme: Internet-facing
oIP address type: IPv4
3.Listeners:
oDefault: HTTP (80)

2.3 Configure Availability Zones
Select the  deafault VPC where your EC2 instances exist
Select two /all subnets (one for each instance if possible)

2.4 Configure Security Groups
Choose the lb-sg security group created earlier

2.5 Configure Target Groups
1.Create a new target group:
oName: web-servers-tg
oTarget type: Instance
oProtocol: HTTP
oPort: 80
oHealth checks: / (default)
2.Register Targets:
oSelect EC2 INSTANCES HERE --- webserver-1 and webserver-2
oClick Include as pending → Register

2.6 Review and Create
Review all settings → Click Create Load Balancer
AWS will take a few minutes to provision the ALB

Step 3: Verify Load Balancer
1.Go to Load Balancers → Select ALB → Description → DNS name
2.Copy the DNS name (something like my-alb-123456.us-east-1.elb.amazonaws.com)
3.Paste it in a browser → You should see either WebServer-1 or WebServer-2 page.
oRefresh multiple times → Load balancer distributes traffic between the two instances

Step 4: Optional – Test Health Checks
Go to Target Groups → web-servers-tg → Targets
Ensure Status of both instances is healthy
If a server is unhealthy, check the bootstrapping script or security group rules.

Step-by-Step: Auto Scaling Group with Load Balancer

Step 1: Create an AMI from Existing EC2 Instance
1.Go to EC2 Console → Instances
2.Select the EC2 instance you already configured with application
3.Click Actions → Image and Templates → Create Image
4.Provide Image Name: e.g., my-webserver-AMI
5.Optional: Add description
6.Click Create Image
7.Wait for the AMI to become available (status: available in AMIs section)
✅ This AMI will be used for launching new instances automatically in ASG.

Step 2: Create a Launch Template
1.Go to EC2 → instances->Launch Templates → Create Launch Template
2.Launch Template Name: my-launch-template
3.AMI: Select the AMI created in Step 1 (my-webserver-AMI)
4.Instance Type: t2.micro (or your choice)
5.Key Pair: Select your existing key pair
6.Security Groups: Select the Load Balancer Security Group (allow HTTP 80)
7.User Data: Optional (already baked into AMI)
8.Click Create Launch Template
✅ Launch Template is ready for Auto Scaling.

Step 3: Create Auto Scaling Group (ASG)
1.Go to EC2 → Auto Scaling Groups → Create Auto Scaling Group
2.Name: webserver-asg
3.Launch Template: Select my-launch-template
4.Template Version: Select Version 1
5.Click Next

Step 3.1: Network Settings
1.VPC: Default VPC (or your existing VPC)
2.Availability Zones (AZs): Select all subnets (for high availability)
3.Load Balancer: Enable Attach to an existing load balancer
oSelect your existing Load Balancer
4.Health Check: HTTP
oSet Health Check Grace Period: 300 seconds
5.Click Next

Step 3.2: Configure Group Size
1.Desired Capacity: 2 (initial instances)
2.Minimum Capacity: 1
3.Maximum Capacity: 4
4.Click Next

Step 3.3: Review and Create ASG
1.Review all settings
2.Click Create Auto Scaling Group
✅ Auto Scaling Group is now ready and attached to your Load Balancer.

Step 4: Verify Load Balancer and Auto Scaling
1.Go to Load Balancers → Select your Load Balancer
2.Click Instances → Target Group → Registered Targets
oYou will see 2 healthy instances initially
3.Check Auto Scaling Group → Instances
oYou may see up to 4 instances if scaling triggers
4.Test auto recovery:
oTerminate one of the instances
oAfter a few seconds, ASG will launch a new instance automatically
oTraffic will automatically be distributed by the Load Balancer

Step 5: Test the Load Balancer
1.Copy the Load Balancer DNS Name from ALB
2.Open in a browser → You should see your web page from one of the instances
3.Refresh multiple times → traffic alternates between instances
✅ This confirms Auto Scaling + Load Balancer is working correctly.

Summary of Flow
1.Create AMI → snapshot of working EC2
2.Create Launch Template → using AMI, security group
3.Create Auto Scaling Group → attach Launch Template, set min/desired/max, attach Load Balancer
4.Verify instances → ASG launches/replaces instances automatically
5.Test Load Balancer → distributes traffic between all active instances


-----------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------

Elastic beanstalk


WEEK-10--Elasticbeanstalk

Prerequisites
A web application ready (e.g., .war, .zip, or code)

 Steps to Create Elastic Beanstalk Application
Step 1: Open Elastic Beanstalk
1.Login to AWS Console
2.Under Compute section ->Search for Elastic Beanstalk
3.Click Create Application

 Step 2: Create Application
1.Enter Application Name (e.g., MyApp)
2.Add description (optional)
3.Click Create

 Step 3: Create Environment
1.Click Create Environment
2.Choose:
o Web Server Environment (for web apps)
3.Click Select

 Step 4: Configure Environment
1.Enter Environment Name
2.Choose Platform:
otomcat (for .war)
otomcat     /Node.js / Python / PHP / etc.
3.Under Application Code:
oSelect Upload your code
oUpload your .war or .zip file

 Step 5: Configure Service Access
1.Service Role → Select existing  -LabRole
2.EC2 Key Pair → Optional (for SSH)
3.EC2 Instance Profile → select (e.g., LabInstanceProfile)

 Step 6: Configure Network
1.Select VPC (default is fine)
2.Select Subnets
3.Enable Public IP

 Step 7: Instance & Scaling Setup
1.Choose Instance type (e.g., t2.micro – free tier)
2.Set:
oMin instances: 1
oMax instances: 2 (or more)
3.Load Balancer → Enabled (optional but recommended)

 Step 8: Review & Create
1.Click Review
2.Click Create

 Deployment Process
AWS automatically creates:
oEC2 instances
oLoad Balancer
oAuto Scaling Group
oSecurity Groups

 Access Your Application
After deployment:
oYou will get a URL (domain link)
oOpen it in browser → Your app will run

Updating Application
1.Go to your environment
2.Click Upload and Deploy
3.Upload new version


------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------


SNS SQS LAMBDA




WEEK-8---SNS,SQS & S3-SNS-SQS-LAMBDA SERVICES
1) Simple Notification Sevice (SNS)
A) Create SNS Topic and Send Message to Email
We will:
1.Create an SNS Topic
2.Create an Email Subscription
3.Confirm the subscription
4.Publish a test message

Step 1: Create SNS Topic
1.Login to AWS Console
2.In search bar → Type SNS
3.Open Amazon SNS (Simple Notification Service)
4.In left panel → Click Topics
5.Click Create topic
Configure:
Type → Select Standard
Name → MyEmailTopic
Leave other settings default (unless required)
6.Click Create topic

 Step 2: Create Email Subscription
1.Open your created topic
2.Go to Subscriptions tab
3.Click Create subscription
Configure:
Protocol → Select Email
Endpoint → Enter your email address
4.Click Create subscription

 Step 3: Confirm Subscription
Go to your email inbox
You will receive a mail from Amazon SNS
Click Confirm subscription
Now your email is successfully subscribed 🎉

 Step 4: Publish Test Message
1.Open your SNS topic
2.Click Publish message
3.Add:
oSubject → Test Mail
oMessage body → Hello this is SNS test
4.Click Publish message
 You will receive email notification.

 B) Trigger Email When S3 Object is Uploaded (S3 → SNS → Email)
Now we extend this setup:
Flow:
S3 Bucket → Event Notification → SNS Topic → Email

 Step 1: Create S3 Bucket
1.Search → S3
2.Click Create bucket
Configure:
Bucket name → my-upload-bucket-2026
Choose Region (Important: Same region as SNS topic)
Keep default settings (unless specific requirement)
3.Click Create bucket

 Step 2: Create SNS Topic (If not already created)
You can reuse the topic from Part A
OR create new topic like:
Topic name → S3UploadNotification
Make sure email subscription is confirmed.

 Step 3: Allow S3 to Publish to SNS (Access Policy)
1.Open SNS Topic
2.Go to Access policy
3.Click Edit
Add permission for S3 to publish.
If using Basic Editor:
Access Policy that needs to be configured on SNS:
a) replace SNS topic ARN
b) Replace s3 bucket ARN
 c) Replace account
{
  "Version": "2012-10-17",
  "Id": "example-ID",
  "Statement": [
    {
      "Sid": "Example SNS topic policy",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": [
        "SNS:Publish"
      ],
      "Resource": "SNS topic ARN",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "s3 bucket ARN"
        },
        "StringEquals": {
          "aws:SourceAccount": "account id"
        }
      }
    }
  ]
}

(Modern AWS console often auto-generates this when configuring from S3)

 Step 4: Configure S3 Event Notification
1.Open your S3 bucket
2.Go to Properties
3.Scroll to Event notifications
4.Click Create event notification
Configure:
Event name → UploadNotification
Event types → Select:
o All object create events
Destination:
oSelect SNS topic
oChoose your topic (S3UploadNotification)
5.Click Save changes

 Step 5: Test It
1.Upload any file into the S3 bucket
2.After upload completes:
oS3 triggers event
oSNS receives publish request
oSNS sends email
 You will receive mail about object upload.

Simple Queue Service (SQS)

2) Create SQS Queue and Send Message
We will:
1.Create an SQS Queue
2.Send a message to the Queue
3.Receive the message from the Queue

Step 1: Create SQS Queue
1.Login to AWS Console
2.In the search bar → Type SQS
3.Open Amazon Simple Queue Service
4.In the dashboard → Click Create queue
Configure:
• Type → Select Standard
• Name → MyQueue
• Leave other settings as default
5.Click Create queue
Your SQS queue is now created successfully.

Step 2: Send Message to Queue
1.Open the MyQueue queue
2.Click Send and receive messages
3.In the Message body, type:
Hello this is SQS test message
4.Click Send message
Your message is now stored in the queue.

Step 3: Receive Message from Queue
1.In the same page (Send and receive messages)
2.Click Poll for messages
You will see the message displayed.
Example output:
Hello this is SQS test message
3.After processing, click Delete message (optional)

 Result:
SQS queue created
Message sent successfully
Message received from queue


3) S3 Bucket Event → SNS → SQS → Lambda Processing
S3 bucket event (like file upload) triggers SNS, which sends the notification to SQS. Then Lambda acts as the consumer and processes the message.
Services used:
• Amazon S3
• Amazon Simple Notification Service
• Amazon Simple Queue Service
• AWS Lambda

S3 → SNS → SQS → Lambda Architecture
S3 Bucket (File Upload)
        ↓
     SNS Topic
        ↓
     SQS Queue
        ↓
   Lambda Function (Consumer)
When a file is uploaded, S3 sends an event notification to SNS, which forwards the message to SQS. The Lambda function then reads and processes the message from SQS.

Step 1: Create S3 Bucket
1.Login to AWS Console
2.Search S3
3.Open Amazon S3
4.Click Create bucket
Configure:
• Bucket name → mys3eventbucket123 (must be unique)
• Region → choose your region
Leave other settings default.
5.Click Create bucket

Step 2: Create SNS Topic
1.Search SNS in AWS Console
2.Open Amazon SNS
3.Click Topics → Create topic
Configure:
• Type → Standard
• Name → MyS3SNSTopic
4.Click Create topic
Copy the SNS Topic ARN.

Step 3: Create SQS Queue
1.Search SQS in AWS Console
2.Open Amazon SQS
3.Click Create queue
Configure:
• Type → Standard
• Name → MyS3Queue
4.Click Create queue
Copy the Queue ARN.

Step 4: Subscribe SQS to SNS
1.Open MyS3SNSTopic
2.Click Create subscription
Configure:
• Protocol → Amazon SQS
• Endpoint → Select MyS3Queue
3.Click Create subscription

Step 5: Add Access Policy to SQS
Open MyS3Queue → Access Policy → Edit
Replace:
• SQS ARN
• SNS ARN
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "sns.amazonaws.com"
      },
      "Action": "sqs:SendMessage",
      "Resource": "SQS ARN",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "SNS ARN"
        }
      }
    }
  ]
}

(Modern AWS console often auto-generates this when configuring from S3)

Click Save.

Step 6: Add Access Policy to SNS
Open SNS Topic → Edit access policy
Replace:
• SNS topic ARN
• S3 bucket ARN
• Account ID
{
  "Version": "2012-10-17",
  "Id": "example-ID",
  "Statement": [
    {
      "Sid": "Example SNS topic policy",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": [
        "SNS:Publish"
      ],
      "Resource": "SNS topic ARN",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "S3 bucket ARN"
        },
        "StringEquals": {
          "aws:SourceAccount": "Account ID"
        }
      }
    }
  ]
}
Save the policy.

Step 7: Configure S3 Event Notification
1.Open S3 Bucket (mys3eventbucket123)
2.Go to Properties
3.Scroll to Event notifications
Click Create event notification
Configure:
• Event name → S3UploadEvent
• Event type → All object create events
Destination:
• SNS Topic → MyS3SNSTopic
Click Save changes.

Step 8: Test the Architecture
1.Open the S3 bucket
2.Click Upload file
3.Upload any file (example: test.txt)
Now:
File Upload → S3
        ↓
Notification → SNS
        ↓
Message → SQS

Step 9: Verify Message in SQS
1.Open MyS3Queue
2.Click Send and receive messages
3.Click Poll for messages
You will see the S3 event notification message.

Step 10: Configure Lambda as Consumer
Now configure Lambda to automatically process messages from SQS.
Create Lambda Function
1.Search Lambda in AWS Console
2.Open AWS Lambda
3.Click Create Function
Configure:
• Author from scratch
• Function name → SQSConsumerFunction
• Runtime → Python 3.x
Click Create function

Add SQS Trigger
1.Open SQSConsumerFunction
2.Click Add Trigger
3.Select SQS
4.Choose MyS3Queue
Click Add
Now Lambda will automatically read messages from SQS.

Lambda Code
Replace code with:
def lambda_handler(event, context):

    for record in event['Records']:
        print("Message received from SQS:")
        print(record['body'])

    return {
        'statusCode': 200,
        'body': 'Message processed successfully'
    }
Click Deploy.

Final Architecture
User uploads file
        ↓
Amazon S3
        ↓
SNS Topic
        ↓
SQS Queue
        ↓
Lambda Function (Consumer)
        ↓
Processing / Logs / Database

Result
When a file is uploaded to S3:
1.S3 sends a notification to SNS
2.SNS forwards the message to SQS
3.SQS stores the message
4.Lambda automatically reads and processes the message
This creates a serverless event-driven architecture commonly used in real cloud applications.



----------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------

IAM USERS


WEEK-12 –Creating IAM USER with permission  Access it with GUI and CLI
Part A: GUI Access (Management Console)
Objective: Create an IAM user with restricted permissions and verify access via the AWS web portal.
1. Administrative Setup (Root User)
Step 1: Log in to the AWS Management Console as the Root User.
Step 2: Search for IAM under the "Security, Identity, & Compliance" category.
Step 3: Navigate to Users and click Create user.
Step 4: User Details: Enter a name (e.g., S3_Specialist). Check the box for "Provide user access to the AWS Management Console." – custom password.
Step 5: Set Permissions: Select "Attach policies directly." Search for and select AmazonS3FullAccess.
Step 6: Review & Create: Complete the wizard. On the final page, click Download .csv file.
Note: This file contains your Console Password and the unique Sign-in URL.
Step 7: Copy your 12-digit Account ID (found in the top-right dropdown or the .csv) and Sign Out.
2. Verification (IAM User Login)
Step 8: Use the Sign-in URL from the .csv (or go to the AWS sign-in page and select IAM User).
Step 9: Enter the Account ID, then the IAM Username and Password from the .csv.
Step 10: The Logic Test:
oTest 1 (Failure): Navigate to EC2. Try to view instances. You will see "API Error" or "Access Denied." This proves the user is restricted.
oTest 2 (Success): Navigate to S3. Create a new bucket. It will work perfectly because of the attached policy.

Part B: CLI Access (Programmatic Access)
Objective: Access AWS resources from a standalone system (CMD/Terminal) using security keys.
1. Credential Generation
Step 1: While logged in as an Admin/Root, go to IAM > Users and click on your user.
Step 2: Go to the Security credentials tab. Scroll to Access keys and click Create access key.
Step 3: Select Command Line Interface (CLI). Acknowledge the warning and click Next.
Step 4: Download the .csv file. > Critical: This contains the Access Key ID and Secret Access Key. If lost, you must delete and recreate them.
2. Standalone System Configuration
Step 5: Install the AWS CLI Tool  from google on your computer.
Step 6: Open your Command Prompt (CMD) or Terminal.
Step 7: Type aws configure and press Enter.
Step 8: Provide the following from your .csv:
oAccess Key ID
oSecret Access Key
oDefault region: (e.g., ap-south-1)
oDefault output: json
3. Verification via Commands
Step 9: Run aws s3 ls. This should list your buckets.
Step 10: Run aws s3 mb s3://your-unique-bucket-name. (Success)
Step 11: Run aws iam list-users.
oResult: You will get an (AccessDenied) error.
This confirms that the CLI is respecting the same IAM policy as the GUI.

-----------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------------------


IAM ROLES


Procedure: Attach an IAM Role to an EC2 Instance (S3 access without keys)
Step 1 — Create Role in Amazon Web Services IAM
1.Open IAM Console → Roles → Create role
2.Trusted entity: Select AWS Service
3.Use case: Choose EC2
4.Click Next
Step 2 — Add Permissions
1.Search policy: AmazonS3FullAccess
2.Select it → Next
Step 3 — Name the Role
Name: EC2-S3-Role
Click Create role
Step 4 — Attach Role to EC2 Instance
1.Go to EC2 Dashboard
2.Select running instance
3.Actions → Security → Modify IAM role
4.Select EC2-S3-Role → Update
Step 5 — Verify from Instance (No Access Keys!)
SSH into instance and run:
aws s3 ls
aws ec2 describe-instances
aws s3 ls  Works
aws ec2 describe-instances Unauthorized (no EC2 permission)
 Reason: Instance receives temporary credentials from IAM Role automatically.
