
# 🚀 Terraform Project: Launch 2 Ubuntu EC2 Instances with Apache in AWS

This Terraform project launches **two Ubuntu EC2 instances** in the **ap-south-1 (Mumbai)** AWS region with:
- Apache installed via `user_data`
- Elastic IPs associated
- Security groups for SSH (port 22) and HTTP (port 80)
- Uses variables for region, instance type, etc.
- Outputs public IPs of instances

---

## 📁 Project Structure

```bash
.
├── main.tf          # Main Terraform configuration
├── variables.tf     # Input variables
├── outputs.tf       # Output definitions
└── README.md        # This documentation
```

---

## 🔧 Prerequisites

- AWS account with Access Key and Secret Key
- Existing **EC2 Key Pair** named `Terraform` in region `ap-south-1`
- [Terraform](https://learn.hashicorp.com/terraform) installed
- Optional: AWS CLI installed and configured

---

## 📄 Terraform Configuration

### `main.tf`

```hcl
provider "aws" {
  region     = var.region
  access_key = "<Your_Acces_key>"
  secret_key = "<Your_Secret_Key>"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_security_group" "ssh" {
  name        = "allow_ssh"
  description = "Allow SSH inbound traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "http" {
  name        = "allow_http"
  description = "Allow HTTP inbound traffic"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  count         = 2
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  key_name      = var.key_name

  vpc_security_group_ids = [
    aws_security_group.ssh.id,
    aws_security_group.http.id,
  ]

  tags = {
    Name        = "WebServer-${count.index + 1}"
    Environment = var.environment
  }

  user_data = <<-EOF
              #!/bin/bash
              apt update -y
              apt install apache2 -y
              systemctl start apache2
              systemctl enable apache2
              echo "Hello from Ubuntu EC2 instance ${count.index + 1}" > /var/www/html/index.html
              EOF

  root_block_device {
    volume_size = var.ebs_volume_size
    volume_type = "gp2"
  }
}

resource "aws_eip" "web_ip" {
  count    = 2
  instance = aws_instance.web[count.index].id
}
```

---

### `variables.tf`

```hcl
variable "region" {
  default = "ap-south-1"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "key_name" {
  default = "<Your_Key_Name>"
}

variable "environment" {
  default = "dev"
}

variable "ebs_volume_size" {
  default = 10
}
```

---

### `outputs.tf`

```hcl
output "ec2_public_ips" {
  description = "Public IPs of EC2 instances"
  value       = aws_eip.web_ip[*].public_ip
}
```

---

## 🚀 Terraform Commands

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

---

## 🌐 Result

- Two EC2 instances with Apache installed
- Elastic IPs automatically assigned
- Access via web browser (HTTP) and SSH
- Outputs public IPs

---

## 🧹 To Destroy Resources

```bash
terraform destroy -auto-approve
```

--- 

## 📌 Note

Do **NOT** expose your access keys in production. Use environment variables or AWS profiles instead.

