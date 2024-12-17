# terraform-aws-ec2-vscode
# Terraform with AWS EC2 Setup - README

## Overview

This README provides a step-by-step guide to deploying an **EC2 instance** on **AWS** using **Terraform** with **VS Code**. By following these instructions, you'll be able to configure AWS credentials, write Terraform configuration files, and deploy an EC2 instance in the **Ireland region** (`eu-west-1`).

---

## Prerequisites

Before you begin, ensure you have the following tools installed on your local machine:

- **[VS Code](https://code.visualstudio.com/)** (with the HashiCorp Terraform plugin)
- **[Terraform](https://www.terraform.io/downloads.html)**
- **[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)**
- An **AWS account** with appropriate permissions to create EC2 instances, VPCs, and security groups.

---

## Steps to Set Up

### 1. Install Terraform Extension in VS Code
Install the **HashiCorp Terraform** plugin in VS Code for syntax highlighting, auto-completion, and other features that enhance your Terraform workflow.

---

### 2. Configure AWS Provider in Terraform

Create a new file `terraform.tf` and define the AWS provider:

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-west-1"  # Ireland region
}
```

---

### 3. Configure EC2 Instance with Terraform

Create another file `ec2.tf` for configuring the EC2 instance, security group, and key pair:

```hcl
# Define SSH Key Pair
resource "aws_key_pair" "terra-key-vscode" {
  key_name   = "junoon-key-vscode"
  public_key = file("junoon-key-vscode.pub")  # Provide the public key file path
}

# Define Default VPC
resource "aws_default_vpc" "default" {}

# Define Security Group
resource "aws_security_group" "my_sg" {
  name        = "my-zplus-security"
  description = "Security group for EC2 instance"
  vpc_id      = aws_default_vpc.default.id

  ingress {
    description = "Allow SSH access on port 22"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow HTTP access on port 80"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outgoing traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "my-zplus-sec"
  }
}

# Define EC2 Instance
resource "aws_instance" "my_instance" {
  ami           = "ami-0e9085e60087ce171"  # Replace with a valid AMI ID for the Ireland region
  instance_type = "t2.micro"
  security_groups = [aws_security_group.my_sg.name]
  key_name        = aws_key_pair.terra-key-vscode.key_name

  root_block_device {
    volume_size = 10
    volume_type = "gp3"
  }
}
```

Make sure to replace the `ami-0e9085e60087ce171` with the correct **AMI ID** for your desired region.

---

### 4. Set Up AWS CLI

1. Install the **AWS CLI** on your machine if not already installed.
2. Run the following command to configure AWS CLI with your credentials:

   ```bash
   aws configure
   ```

   You'll be prompted to enter your **Access Key**, **Secret Access Key**, **Region**, and **Output format**. The `Access Key` and `Secret Access Key` can be obtained from your AWS account under **IAM > Security Credentials**.

---

### 5. Initialize Terraform and Apply Configuration

1. **Initialize Terraform**:

   In your terminal, navigate to the folder where your Terraform files (`terraform.tf` and `ec2.tf`) are located, then run:

   ```bash
   terraform init
   ```

2. **Check the plan**:

   Verify the changes Terraform will make by running:

   ```bash
   terraform plan
   ```

3. **Apply the configuration**:

   Apply the configuration to create the resources on AWS:

   ```bash
   terraform apply
   ```

   Terraform will prompt you to confirm the changes. Type `yes` to proceed.

---

### 6. Verify EC2 Instance Creation

Once the process is completed, Terraform will output the creation of your EC2 instance in the **Ireland region** (`eu-west-1`). You can go to the **AWS Management Console** to confirm that the EC2 instance has been created.

---

### 7. SSH into Your EC2 Instance

1. **Revoke inheritance and grant proper permissions to your SSH key** (replace `junoon-key-vscode` with your key file name):

   ```bash
   icacls junoon-key-vscode /inheritance:r /grant:r "DELL:R"
   ```

2. **SSH into the EC2 instance**:

   Use the SSH client and your private key to connect:

   ```bash
   ssh -i "junoon-key-vscode.pem" ubuntu@ec2-34-255-7-235.eu-west-1.compute.amazonaws.com
   ```

   Replace `junoon-key-vscode.pem` with your private key file and update the `ec2-34-255-7-235.eu-west-1.compute.amazonaws.com` with your actual EC2 instance's public IP address.

---

## Conclusion

Congratulations! You've successfully set up and deployed an EC2 instance using **Terraform** on **AWS** with **VS Code**. Terraform makes it easy to automate infrastructure provisioning, allowing you to manage your cloud resources as code.

---

## Useful Links

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [Terraform Getting Started](https://learn.hashicorp.com/terraform)

---

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.
