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