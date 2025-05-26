# AWS VPC Infrastructure with SAM

This project provides an AWS Serverless Application Model (SAM) template to provision a Virtual Private Cloud (VPC) with a standard networking setup. This includes public and private subnets across two Availability Zones, an Internet Gateway for public subnet outbound access, and a NAT Gateway for private subnet outbound access.

While SAM is often associated with serverless functions and APIs, it is built on AWS CloudFormation. This means you can use SAM to deploy and manage any AWS resources that CloudFormation supports, including core networking infrastructure like VPCs, as demonstrated in this template.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Template Overview](#template-overview)
- [Deployment Instructions (Using SAM CLI)](#deployment-instructions-using-sam-cli)
  - [1. Save the Template](#1-save-the-template)
  - [2. Package the Template](#2-package-the-template)
  - [3. Deploy the Stack](#3-deploy-the-stack)
- [Key Resources Created](#key-resources-created)
- [Stack Outputs](#stack-outputs)
- [Cleanup (Using SAM CLI)](#cleanup-using-sam-cli)

## Prerequisites

Before you begin, ensure you have the following:

1.  **AWS Account**: An active AWS account.
2.  **AWS CLI**: The [AWS Command Line Interface](https://aws.amazon.com/cli/) installed and configured with credentials.
    * You can configure it by running `aws configure`. SAM CLI uses these credentials.
3.  **AWS SAM CLI**: The [AWS Serverless Application Model CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) installed.
4.  **S3 Bucket (for packaging)**: An S3 bucket in the region where you intend to deploy. This is used by the `sam package` command.

## Template Overview

The `template.yaml` file defines the following core AWS networking resources using CloudFormation syntax, which is interpreted by SAM:

* **VPC**: A dedicated Virtual Private Cloud (`CustomTestVPC`) with a CIDR block of `18.0.0.0/20`.
* **Subnets**:
    * Two Public Subnets (`PublicSubnetOne`, `PublicSubnetTwo`) in different Availability Zones (`us-east-1a`, `us-east-1b` by default). Instances in these subnets can have public IP addresses and direct internet access.
    * Two Private Subnets (`PrivateSubnetOne`, `PrivateSubnetTwo`) in different Availability Zones. Instances in these subnets do not have direct internet access but can access the internet via the NAT Gateway.
* **Internet Gateway (IGW)**: Attached to the VPC to allow communication between instances in the VPC and the internet.
* **Route Tables**:
    * A Public Route Table associated with the public subnets, routing internet-bound traffic (`0.0.0.0/0`) to the IGW.
    * A Private Route Table associated with the private subnets, routing internet-bound traffic (`0.0.0.0/0`) to the NAT Gateway.
* **Elastic IP (EIP)**: A static public IP address for the NAT Gateway.
* **NAT Gateway**: Deployed in one of the public subnets (`PublicSubnetOne`) to enable instances in the private subnets to initiate outbound traffic to the internet or other AWS services, while preventing the internet from initiating connections with those instances.

## Deployment Instructions (Using SAM CLI)

Follow these steps to deploy the VPC infrastructure using the AWS SAM CLI:

### 1. Save the Template

Ensure the SAM template code (provided in previous discussions or as part of your project) is saved as `template.yaml` in your project directory.

### 2. Package the Template

The `sam package` command uploads your template and any local artifacts (though this template primarily defines AWS resources directly) to an S3 bucket. This packaged template is then used for deployment.

```bash
sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket YOUR_S3_BUCKET_NAME # Replace with your S3 bucket name
```
Replace YOUR_S3_BUCKET_NAME with the name of an S3 bucket in your desired deployment region.

### 3. Deploy the Stack
Use the sam deploy command to deploy your packaged AWS resources. The --guided flag makes the process interactive.

The template is designed for the us-east-1 region by default. If you need to deploy to another region, ensure you modify the AvailabilityZone properties in template.yaml accordingly and specify the correct region here.

```
sam deploy --guided \
    --template-file packaged.yaml \
    --region us-east-1 # Or your target region
```

You will be prompted for:

- Stack Name: A unique name for your CloudFormation stack (e.g., my-custom-vpc-sam-stack).

- AWS Region: Confirm or enter the region (e.g., us-east-1). Ensure this matches the Availability Zones in the template or that the template is modified for this region.

- Parameter Username [admin]: (This prompt may appear if your samconfig.toml or template has parameters defined. For this specific VPC template, there are no custom parameters by default, so you might not see this unless your samconfig.toml implies it).

- Confirm changes before deploy: Review changes before applying (y/n).

- Allow SAM CLI IAM role creation: Usually y. (SAM may create a role to manage CloudFormation changes).

- Disable rollback: n (Recommended to roll back on failure).

- Save arguments to samconfig.toml: y (Convenient for future deployments, creates a samconfig.toml file).

- Capabilities: You might be asked to acknowledge capabilities if the stack creates IAM resources or macros. For this VPC, if prompted for capabilities like CAPABILITY_IAM or CAPABILITY_NAMED_IAM, it's safe to acknowledge.

After deployment, your VPC and related resources will be active.

### Key Resources Created
1 x VPC
2 x Public Subnets
2 x Private Subnets
1 x Internet Gateway
1 x Elastic IP (for NAT Gateway)
1 x NAT Gateway
2 x Route Tables (1 public, 1 private)
Various Route Table Associations and Routes


### Stack Outputs
The CloudFormation stack managed by SAM will provide the following outputs upon successful creation:

- **VPCId:** The ID of the created VPC.
- PublicSubnetOneId: The ID of the first public subnet.
- PublicSubnetTwoId: The ID of the second public subnet.
- PrivateSubnetOneId: The ID of the first private subnet.
- PrivateSubnetTwoId: The ID of the second private subnet.
- NatGatewayEIPAddress: The public IP address of the NAT Gateway's EIP.

## Cleanup (Using SAM CLI)
To remove all the resources created by this template, you can delete the CloudFormation stack using the SAM CLI:
```
sam delete --stack-name YOUR_STACK_NAME --region YOUR_DEPLOYMENT_REGION --no-prompts
```