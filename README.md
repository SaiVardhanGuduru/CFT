# DevOps EC2 CloudFormation Template  
Provision an EC2 instance for DevOps practice with Terraform pre-installed and an auto-mounted 30GB data volume.

---

## 📝 Overview

This repository contains an AWS CloudFormation Template (CFT) that deploys an EC2 instance configured for DevOps learning and experimentation.  
The template automatically:

- Creates a **t3.medium EC2 instance**
- Installs **Terraform**
- Attaches a **30GB EBS volume**
- Automatically detects the correct NVMe device
- Formats it using **XFS**
- Mounts it permanently at **/data**
- Creates `/data/terraform` with correct user permissions
- Attaches an **IAM Role** with AdministratorAccess
- Enables SSH access (port 22)

This setup provides a clean, reusable DevOps lab environment.

---

## 🚀 Features

### **✔ EC2 instance with Terraform**
Automatically installs Terraform via HashiCorp repo.

### **✔ 30GB EBS data volume**
CloudFormation attaches a separate EBS volume exclusively for your work.

### **✔ Auto-mount at /data (persistent)**
A userdata script:
- Detects the NVMe device
- Formats it with XFS (only if needed)
- Adds a UUID-based entry to `/etc/fstab`
- Mounts it at `/data`

### **✔ Terraform workspace ready**
The template automatically creates:

/data/terraform


with correct ownership for `ec2-user`.

### ✔ IAM role with AdministratorAccess

### ✔ Security group allowing SSH access

---

## 🏗 Architecture Diagram (Conceptual)

+---------------------------------------------+
| AWS EC2 |
| |
| +-------------------+ |
| | Root Volume 20GB | --> OS + LVM |
| +-------------------+ |
| |
| +--------------------------+ |
| | Extra EBS Volume 30GB |---> Mounted at /data
| +--------------------------+ |
| |
| Installed: Terraform |
| IAM Role: AdministratorAccess |
| Security Group: SSH (22) |
+---------------------------------------------+

yaml


---

## 📦 Files in this Repository

.
├── template.yaml # The main CloudFormation template
└── README.md # Documentation (this file)

yaml


---

## 🛠 Prerequisites

Before deploying:

- An **existing VPC**
- An **existing public subnet**
- AWS CLI installed (optional)
- IAM permissions to deploy CloudFormation

---

## 🚀 Deployment Instructions

### **1. Deploy via AWS Console**
1. Open **CloudFormation Console**
2. Click **Create stack → With new resources**
3. Upload the `template.yaml`
4. Provide:
   - VPC ID
   - Public Subnet ID
5. Launch the stack
6. Wait until **CREATE_COMPLETE**

---

### **2. Deploy via AWS CLI**

```sh
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name devops-lab \
  --capabilities CAPABILITY_IAM
🗂 After Deployment
SSH into the instance:

sh

ssh -i your-key.pem ec2-user@<public-ip>
Verify mounted volume:

sh

lsblk
df -h /data
Expected:

bash

/dev/nvme1n1   30G   mounted on /data
Terraform workspace:

sh

cd /data/terraform
terraform -version
❗ Understanding the Storage Layout
This AMI may internally expose multiple NVMe devices.
The userdata script automatically detects the correct extra EBS volume, formats it, and mounts it at /data.

You do not need to manually handle:

/dev/nvme0n1

/dev/nvme1n1

/dev/nvme2n1

Everything is automated.

🧪 Testing Terraform
Inside the instance:

sh

cd /data/terraform

cat <<EOF > main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
EOF

terraform init
terraform plan
🧩 Troubleshooting
1. /data not mounted
Run:

sh

sudo mount -a
sudo tail -50 /var/log/cloud-init-output.log
2. Device detection failed
The CFT uses EC2 metadata + NVMe fallback logic.
If volume is not found, ensure stack logs show no userdata errors.

📌 Notes
This template is built for learning and DevOps practice, not production use.

You can safely modify instance type, volume size, or tools installed.

The IAM role grants AdministratorAccess for convenience.

🤝 Contributions
Feel free to open PRs to:

Add more DevOps tools (Docker, kubectl, helm)

Add logging (CloudWatch)

Improve the userdata script

📄 License
This project is open-source and free to use.
