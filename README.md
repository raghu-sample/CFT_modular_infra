Project 1: Modular Networking + Compute

 Introduction
 This document provides an overview and step-by-step guide for a two-stack AWS CloudFormation project:
 1. Network Stack: VPC with public/private subnets, IGW, NAT Gateway, and route tables.
 2. Compute Stack: ALB, Target Group, Listener, Launch Template, and Auto Scaling Group.

 Network Stack Template
 `network-stack.yaml`- Defines AWS::EC2::VPC, InternetGateway, Subnets, NAT Gateway, and Route Tables.- 
 
 Exports: VpcId, PublicSubnet1Id, PublicSubnet2Id.
 
 Compute Stack Template
 `compute-stack.yaml`- Creates a Security Group for HTTP traffic.- Provisions an internet-facing 
 
 Application Load Balancer across two subnets.- Defines a Target Group and Listener for HTTP on port 80.- Uses AWS::EC2::LaunchTemplate for EC2 instance configuration.- Deploys an Auto Scaling Group referencing the Launch Template.
 
 Deployment Steps
 1. Validate and deploy Network Stack:
   aws cloudformation validate-template --template-body file://network-stack.yaml
   aws cloudformation deploy --template-file network-stack.yaml --stack-name network-stack --capabilities CAPABILITY_NAMED_IAM
 2. Validate and deploy Compute Stack:
   aws cloudformation validate-template --template-body file://compute-stack.yaml
   aws cloudformation deploy --template-file compute-stack.yaml --stack-name compute-stack --parameter-overrides KeyName=<YourKeyPairName> --capabilities CAPABILITY_NAMED_IAM
 
 Verification
 Retrieve ALB DNS:
   aws cloudformation describe-stacks --stack-name compute-stack --query "Stacks[0].Outputs[?OutputKey=='LoadBalancerDNS'].OutputValue" --output text
 Test endpoint:
   curl http://<ALB_DNS>/ => "Hello from <hostname>"
 Cleanup
 Delete Compute and Network stacks:
   aws cloudformation delete-stack --stack-name compute-stack
   aws cloudformation delete-stack --stack-name network-stack
 Next Steps- Enable drift detection in the AWS Console.- Add Auto Scaling policies using AWS::AutoScaling::ScalingPolicy.- Integrate CloudWatch alarms and SNS notifications.
- Modularize further with nested stacks.- Explore serverless projects with AWS SAM


-- basic understanding:
why launch config 

Defines how each EC2 instance in your ASG should be launched:

AMI & instance type

SSH key pair from KeyName

Attach the SG

UserData bootstraps an HTTP server and sets a simple welcome page.

Docs lookup

LaunchConfig: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-launchconfig.html

Aim of compute stack :
The Compute stack’s primary aim is to stand up and manage your application’s compute layer—separate from networking—by provisioning:

Security Perimeter

A Security Group that allows only HTTP (port 80) inbound traffic from the internet.

Load Balancer

An internet‑facing Application Load Balancer (ALB) across two public subnets for high availability.

A Target Group and Listener that route HTTP requests to healthy back‑end instances.

Instance Configuration

An EC2 Launch Template that defines the AMI, instance type, SSH key, security group, and startup script (UserData) to install and serve a simple web page.

Auto Scaling

An Auto Scaling Group (ASG) that:

Launches a baseline of 2 instances (scaling up to 4 if you add policies later)

Registers them with the Target Group so the ALB can distribute traffic

Automatically replaces any unhealthy instances




******
By breaking out all of these resources into a dedicated “compute” stack—even though they reference the VPC and subnets exported by your Network stack—you’re achieving:

Modularity & Reuse: You can update or replace your compute layer without touching your networking

Scalability & Fault‑Tolerance: The ALB + ASG pattern ensures traffic is balanced and capacity adjusts to load

Infrastructure as Code: Everything is version‑controlled and repeatable via CloudFormation

In short, the Compute stack turns your raw network fabric (VPC + subnets) into a fully functional, scalable web‑serving environment that you can deploy, update, and tear down with a single CLI command.