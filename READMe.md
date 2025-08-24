# aws-cloudformation-ec2-public-stack-deploy

## Introduction

With the growing preference for **low-code and no-code** solutions, many businesses want to automate infrastructure without deep DevOps expertise. **AWS CloudFormation** meets this need by being **natively integrated into AWS**, requiring no extra tooling, and offering a **declarative, template-driven approach** that feels more like configuration than coding. 

Compared to third-party tools like **Terraform**, CloudFormation is often **faster to adopt** and helps reduce both setup time and operational overhead. This makes it an attractive option for organizations prioritizing **speed, simplicity, and automation** without heavy engineering investment.  

This repository demonstrates the **power and flexibility of CloudFormation** by deploying a secure, highly available application stack on AWS.  


## Problem Statement  

Design and deploy, using **AWS CloudFormation**, a **highly available and secure application environment** on AWS. The solution should include:  

- A **highly available VPC architecture** spanning multiple Availability Zones.  
- Appropriate **networking components** (such as Internet/NAT gateways and routing) to enable both public access and controlled private connectivity.  
- An **EC2 instance** that is automatically bootstrapped to serve the **2048 web game**, with persistent storage provided by **Amazon EFS**.  
- An **Application Load Balancer (ALB)** to distribute traffic across the application layer.  
- An **AWS WAF WebACL** in front of the ALB to protect against common web threats.  

The CloudFormation template should provision all resources and output the ALB’s public DNS name for accessing the application.  


## Solution

This solution is delivered as a single AWS **CloudFormation** template that provisions a secure, highly available web stack and boots the EC2 host with the **2048** game.

### What the stack creates
- **Networking (HA VPC):**
  - One VPC (`VpcCIDR`) with **two AZs**
  - **Public subnets (x2)** and **private subnets (x2)**
  - **Internet Gateway** + public route table with default route to IGW
  - **Two NAT Gateways** (one per AZ) + private route tables with default routes to NATs
- **Security Groups:**
  - **ALB SG** – inbound 80/443 from the internet
  - **EC2 SG** – inbound 22 from the world (tighten in prod), HTTP/HTTPS **only from ALB**
  - **EFS SG** – NFS (2049) from EC2 SG
- **Compute & Storage:**
  - **EC2 instance** (Amazon Linux, `t3.small`) in Public Subnet 1
  - **UserData** bootstraps NGINX and deploys the **2048 web game**
  - **Amazon EFS** file system with a mount target in a private subnet
  - **IAM role/profile** with SSM and EFS permissions
- **Load Balancing & Protection:**
  - **Application Load Balancer** spanning both public subnets
  - **HTTP listener (80)** forwarding to a target group
  - **AWS WAFv2 WebACL** (CommonRuleSet) associated to the ALB

### How this satisfies the problem
- **Highly available VPC:** Multi-AZ public/private subnets, dual NATs, ALB across both public subnets.
- **Controlled connectivity:** IGW for ingress/egress on public, NAT for private egress.
- **EC2 bootstrapped with 2048:** UserData installs NGINX, downloads and serves the **2048** game.
- **Persistent storage:** EFS created and (security-wise) allowed from EC2, ready for mounting.
- **Edge protection:** ALB fronted by **AWS WAF** using managed rules.

### Deploy (CLI)
```bash
aws cloudformation deploy \
  --stack-name demo-ec2-alb-waf-efs \
  --template-file template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    EnvironmentName=demo-ec2 \
    KeyPairName=demo-ec2-keypair \
    VpcCIDR=10.192.0.0/16 \
    Ec2Ami=ami-09b024e886d7bbe74
