Host and Deploy Dynamic Web Application on AWS with Terraform, Docker, Amazon ECR, and Amazon ECS.





# Generating Key Pair

In this lesson we will generate a key-pair on our computer so we can use the keypair to clone our private Github repository.

To do this open Powershell. The command in powehsell that we will type is "ssh-keygen -t rsa -b 2048" This is the command we will use to create a keypair on our computer. Than press enter. It will generate the public and private key. Once you do this it will ask us where we want to save our keypair too. Leave it in the "admin" directory it should be the default one it gives you according to best practice. Press enter until it finishes. It should be saved in the admin folder in the drive. Under jonathan and in the ssh folder. The file is saved as id_rsa.


# Adding the Public SSH Key to your Github Account

Open your github account, go to settings and on the left hand side click the "SSH and GPG keys" tab. Than where it says "New SSH KEY" click that to add the public key that we had just got from powershell. The id_rsa key.. We're going to name the new key as "my-public-key" than copy the key and paste the WHOLE THING. Now that we have done that we are now able to clone our GitHub repository to our computer. 


# Creating IAM User 

Go to IAM and select create user and give it the name "terraform-user" than press next. Now we're going to attach a policy directly to the user. So click "attach policies directly" We're going to give this user "administrator access" so select it. Then click next and create the user. 


# How to generate an Access Key for an IAM user on AWS.

Now we're going to generate an access key for the IAM role we had created for the IAM user we created in the step above. For the user to have programmatic access we have to generate an access key and secret access key for our user. To do this select the user. (terraform-user) and click the security credentials tab below select create access keys. Select the CLI option. Than click create access key and make sure to download both access key and secret access key as a csv file. We are going to use these keys later to be able to authroize and authenticate our AWS credentials once we start using terraform. 

# AWS Configuration: Running the AWS Configure Command 

We will create a named profile for the IAM user we created in the previous steps. Creating a named profile for the IAM user will allow terraform to use that user's credentials to authenticate with our AWS environment. So now open the command prompt. We will use the command prompt to create a named profile for our IAM user. This is the command:
"aws configure --profile terraform-user" press enter than it will ask for the AWS Access key and the secret access key for that user. These are keys we downloaded in a csv file. Copy and paste them in the command prompt as needed. The region will be: "us-east-1" press enter. To see where the iam users credentials are stored on the computer it is stored in the admin folder in a folder titled .aws 


# Storing Terraform state with S3 bucket

In this step we will create a state file that we will store in our S3 bucket. We will first create our S3 bucket inside of AWS. Click "create bucket" and we're going to name it: "jon-terraform-remote-state" and under "bucket versioning" we're going to click enable. Than click create bucket. This is the bucket where we will store our terraform state file. 

Side Note: When you use terraform to create resources in AWS terraform will record the information about the resources it created in a Terraform State File and the next time you go to update those resources terraform will use the state file to find those resources and update them accordingly. The state file is crucial to our terraform works. 


# Create a DynamoDB Table to lock Terraform state 

When you lock your terraform state with DynamoDB this prevents multiple users from making changes to the states at the same time. To do this in the management console on AWS go to DynamoDB in AWS. Select create table, then we're going to give it a name. Name it: "terraform-state-lock" Under partition key type: "LockID" Keep everything else deafult than scroll down and select create table. This is the table we will use for the terraform state lock. 


# Creating AWS Resources with Terraform Syntax


In this example we will see how we can create a VPC using terraform:


resource "aws_vpc" "main" {
  cidr_block       = "10.0.0.0/16"
  instance_tenancy = "default"

  tags = {
    Name = "main"
  }
}

This terraform documentation was taken from terraforms website, by simply going to google and typing in aws terraform vpc and selecting the basic usage with tags. This is the syntax we can use to build a vpc in our aws account. 



# Git Repository Setup for Terraform Code Storage. 

First thing we should do is go to our GitHub account. Click new repository. Name it "terraform-modules"  The description will be "This is a respository for storing terraform modules". Make it private, check "add a readme" box under gitignore select the drop down and search for "terraform. Than click create respository. Next we will pcik a directory where we will clone this repository. After creating the repository click the code drop down and ssh than copy it. Open powershell and type in: "git clone" than paste the ssh code we had copied from github. Than press enter. We cloned the repository on the c-drive under the admin folder we should see it there. Under "terrafrom-modules"




# Create Terrafrom Module for VPC

Open up Visual studio code and open up the respository we had just cloned called "terraform modules" Once we do this on the left hand side click create new folder we're going to name it: "VPC". This is where we will store the vpc module. Once we create the folder we are going to right click on the vpc folder and select "new file" we are going to create three files in it the frist one will be named: "main.tf" The second file we will create in the vpc is folder will be named: "variables.tf" The third file we will create in the vpc folder will be named: "outputs.tf" These three files is where we will create the vpc module. 

The main.tf file is where we will store the resources for our VPC.
The variables.tf we will use to store our variables.
The outputs.tf we will use it to export the attributes in our vpc that we want to reference in another module. 

Now to begin open the main.tf file and in the link in this lessons description on AOSnotes.com open the link and copy the reference file from github and paste it into the main.tf file. Once we do that the first thing we will build is the vpc. To create the cidr block for the vpc we must create a variable for it. So next select the "variable.tf" file on the left hand side. 

Before we create a variable for the vpc cidr block first lets create these 3 variables. 

The first variable we will create is the variable for the "region". This is the region where we are going to deploy our application in. 

Next we will create another variable called "project_name". This is going to be the name of our project.

The third variable we will create is "environment" this is the environment we want to deploy the application in. 

So in the variables.tf file type in a note called "#environment variables" The first one we will create is the region. So type variable in the variables.tf file and select the block. and in the available "" we are going to enter the name of our variable. "region than bring the bracket all the way back up. do the same for the other two variables we will create.

Next we will create varibles for our vpc so underneath the environement variables create a new note and name it "# vpc variables. The variable we will create will be "vpc_cidr" do as we did for the environment variables. Now that we created the cidr_block variable we will use it to reference the cidr_block in the main.tf file on line 3.

So in the main.tf file on line 3 type: "var.vpc_cidr"
On line 4 for the instance tenancy we will type: "default" in quotes. on line 5 we will type: true 

On line 8 we will type in the first bracket var.project_name and in the second we will put var.environment.


Next we will build the internet gateway. The vpc id will be the same as above. the cidr_block for the public subnet az1 we will have to create a new variable for it in our variables.tf file. In the file we will follow the same steps for the other variables created. so it should read: 
variable "public_subnet_az1_cidr" {} and when we reference it in our main.tf file on line 27 it will read:
var.public_subnet_az1_cidr

The availability zone for the public subnet AZ1 will be what is on line 22 of our main.tf file. so on line 28 it should look like this: 
data.aws_availability_zones.available_zones.names[0]

Line 29 will be true

(Note the tags will be the same for each step.)


Repeat the same steps for the public subnet az1 but this time for the availability zone on line 40 copy what is on line 28 and change the [0] to [1]

Line 41 will be true. 

Next we will create the route table. the vpc_id will remain the same. On line 53 the cidr block will be 0.0.0.0/0 in quotes this will route traffic anywhere on the internet. 

On line 54 the gateway_id will be the internet gateway id that we can get from copying line 13 and pasting it, removing the quotes adding a . and at the end adding .id should look like this:

aws_internet_gateway.internet_gateway.id


You should be able to do the rest as it is a repeat of the steps above!!!

Side Note: for "map_public_ip_on_launch" option if in a private subnet should be false and if in public true

Side note for the availability zones the [0] and [1] represent the AZ where the subnet will be built so the [0] is for anything in the AZ1 and [1] is for anything in AZ2.

Also do not forget to create the variables in the variables.tf file for reference for this lecture this is what it should look like so far:

# environment variables
variable "region" {}
variable "project_name" {}
variable "environment" {}

# vpc variables
variable "vpc_cidr" {}
variable "public_subnet_az1_cidr" {}
variable "public_subnet_az2_cidr" {}
variable "private_app_subnet_az1_cidr" {}
variable "private_app_subnet_az2_cidr" {}
variable "private_data_subnet_az1_cidr" {}
variable "private_data_subnet_az2_cidr" {}


After you've completed it all save all. 

Next we're going to generate the values for outputs. So select the output.tf file. 
Use the link in this lectures description to get the information we need for the output.tf file. 
Copy and paste it into VSC. 

Now in the outputs file we're going to list all the values from our variables.tf and main.tf file as needed.

the last 2 outputs remember. az1 = [0], az2 = [1]


Now we can push the module in our github repository. Commit message will be: created vpc module














