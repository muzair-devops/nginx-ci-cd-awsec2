# AWS EC2 Setup with AWS CLI

This documentation provides step-by-step instructions to create and configure an **Ubuntu 24.04 EC2 instance** on AWS using the CLI.

---

## Prerequisites

1. **Install AWS CLI (on macOS)**
   ```bash
   brew install awscli
   ```

2. **Configure AWS CLI**
   ```bash
   aws configure
   ```
   Provide:
   - AWS Access Key ID
   - AWS Secret Access Key
   - Default region (e.g., `us-east-1`)
   - Output format (`json` or `table`)

3. **IAM User Permissions**
   - Ensure your IAM user has permissions for EC2:
     - `ec2:RunInstances`
     - `ec2:DescribeInstances`
     - `ec2:CreateKeyPair`
     - `ec2:CreateSecurityGroup`
     - `ec2:AuthorizeSecurityGroupIngress`

---

## 🔑 Step 1: Create a Key Pair

```bash
aws ec2 create-key-pair   --key-name cli-user   --query 'KeyMaterial'   --output text   --region us-east-1 > cli-user.pem

chmod 400 cli-user.pem
```

- The `.pem` file is required for SSH login. (for )
- Keep it safe and never share it.

---

## 🔐 Step 2: Create a Security Group

1. Create security group:
   ```bash
   aws ec2 create-security-group      --group-name my-sg      --description "Allow SSH access"      --region us-east-1
   ```

2. Allow SSH (port 22) from your IP:
   ```bash
   aws ec2 authorize-security-group-ingress      --group-name my-sg      --protocol tcp      --port 22      --cidr <YOUR_PUBLIC_IP>/32      --region us-east-1
   ```
   Replace `<YOUR_PUBLIC_IP>` with your machine’s IP (find via https://whatismyip.com).

---

## 🖥️ Step 3: Find the Latest Ubuntu 24.04 AMI

```bash
aws ssm get-parameter   --name /aws/service/canonical/ubuntu/server/24.04/stable/current/amd64/hvm/ebs-gp3/ami-id   --region us-east-1   --query 'Parameter.Value'   --output text
```

This returns the latest **Ubuntu 24.04 AMI ID** for your region.

---

## 🚀 Step 4: Launch an EC2 Instance

```bash
aws ec2 run-instances   --image-id <AMI_ID>   --count 1   --instance-type t2.small   --key-name cli-user   --security-groups my-sg   --associate-public-ip-address   --region us-east-1   --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Ubuntu24-Server}]'
```

- Replace `<AMI_ID>` with the value from **Step 3**.
- The instance will launch with the **Name** tag `Ubuntu24-Server`.

---

## 🌍 Step 5: Get the Public IP of Your Instance

```bash
aws ec2 describe-instances   --filters "Name=instance-state-name,Values=running"   --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]"   --region us-east-1   --output table
```

---

## 🔗 Step 6: Connect via SSH

```bash
ssh -i cli-user.pem ubuntu@<PUBLIC_IP>
```

- Replace `<PUBLIC_IP>` with the value from Step 5.
- For Ubuntu AMIs, the default username is **`ubuntu`**.

---

## Summary

- **Key Pair** → required for SSH access.  
- **Security Group** → must allow SSH (port 22).  
- **Ubuntu 24.04 AMI** → fetched from SSM Parameter Store.  
- **EC2 Instance** → launched with name and public IP.  

