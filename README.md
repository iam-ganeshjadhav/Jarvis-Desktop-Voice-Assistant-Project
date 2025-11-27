# 🤖 Jarvis Desktop Voice Assistant  
### Fully Automated Deployment Using **AWS EC2 + Terraform + Jenkins CI/CD**

---

## 📌 Project Overview

Jarvis is a smart **AI-powered Desktop Voice Assistant** built using Python.  
It can respond to voice commands, search Wikipedia, tell jokes, play music, greet you, take screenshots, and automate daily tasks.

In this project, I extended Jarvis into a real DevOps–ready deployment:

### 🚀 **What I Built**
- Automated infrastructure using **Terraform**
- Continuous deployment using **Jenkins**
- GitHub → Jenkins → EC2 auto-deploy pipeline
- Systemd service to run Jarvis automatically on boot
- *Server-mode Jarvis* for AWS EC2 (headless operation)
- Production folder structure with virtual environment

This makes the project a perfect combination of **Python + Cloud + DevOps**.

---

## 🏗️ System Architecture


- **Terraform** provisions:  
  - EC2 instance  
  - IAM role  
  - Security group  
  - User data for auto-deployment  

- **Jenkins** handles:  
  - Webhooks  
  - Code checkout  
  - Deploying updated files to EC2  
  - Restarting Jarvis automatically  

---

## ✨ Features of Jarvis

### 🗣️ Voice Assistant (Desktop Mode)
- Speaks using pyttsx3  
- Listens via microphone  
- Recognizes commands  
- Plays music  
- Opens websites  
- Takes screenshots  
- Tells jokes  
- Greets based on time  
- Shutdown/restart PC  

### ☁️ Server Mode (AWS EC2)
Since EC2 has no GUI/audio devices:
- Screenshot disabled  
- Music disabled  
- Shutdown disabled  
- Microphone disabled  
- Commands typed through terminal  
- Works in headless mode  
- Still supports:
  - Time  
  - Date  
  - Wikipedia search  
  - Jokes  
  - Greeting  

---

## 🧰 Tech Stack

### **Python Modules**
- pyttsx3  
- speech_recognition  
- wikipedia  
- pyjokes  
- pyautogui (desktop only)  

### **DevOps Tools**
- Terraform  
- Jenkins (Pipeline + Webhook)  
- AWS EC2 (Ubuntu)  
- IAM Roles  
- Systemd service  
- GitHub  

---

## 📂 Project Structure
```
Jarvis-Desktop-Voice-Assistant-Project/
│
├── Jarvis/
│ └── jarvis.py
│
├── requirements.txt
├── jenkinsfile
├── IMG
└── README.md
```

---

## 🚀 Deployment Process

### **1️⃣ Infrastructure (Terraform)**  
Terraform automatically:
- Creates EC2 instance  
- Installs Python, Git, Dependencies  
- Clones the repo into `/opt/jarvis`  
- Creates virtual environment  
- Creates & starts `jarvis.service`
  
**main.tf**
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

resource "aws_security_group" "jarvis_sg" {
  name        = "jarvis-sg"
  description = "Allow SSH and Jarvis port"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 5000
    to_port     = 5000
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

resource "aws_iam_role" "jarvis_role" {
  name = "jarvis-ec2-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "ssm_attach" {
  role       = aws_iam_role.jarvis_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

resource "aws_iam_instance_profile" "jarvis_profile" {
  name = "jarvis-ec2-instance-profile"
  role = aws_iam_role.jarvis_role.name
}

resource "aws_instance" "jarvis_ec2" {
  ami                         = var.ami_id
  instance_type               = var.instance_type
  key_name                    = var.key_name
  subnet_id                   = data.aws_subnets.default.ids[0]
  vpc_security_group_ids      = [aws_security_group.jarvis_sg.id]
  iam_instance_profile        = aws_iam_instance_profile.jarvis_profile.name
  associate_public_ip_address = true

user_data = <<-EOF
#!/bin/bash
apt-get update -y
apt-get install -y git python3 python3-venv python3-pip

# Clean folder
rm -rf /opt/jarvis
mkdir -p /opt/jarvis
chown ubuntu:ubuntu /opt/jarvis

# Clone GitHub repo as ubuntu user
sudo -u ubuntu -H git clone https://github.com/iam-ganeshjadhav/Jarvis-Desktop-Voice-Assistant-Project.git /opt/jarvis

# Create venv and install dependencies
cd /opt/jarvis
sudo -u ubuntu python3 -m venv venv
sudo -u ubuntu bash -c "source venv/bin/activate && pip install --upgrade pip && pip install -r requirements.txt"

# Create Systemd service
cat <<'SERVICE' >/etc/systemd/system/jarvis.service
[Unit]
Description=Jarvis Desktop Voice Assistant
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/opt/jarvis
ExecStart=/opt/jarvis/venv/bin/python /opt/jarvis/Jarvis/jarvis.py
Restart=always

[Install]
WantedBy=multi-user.target
SERVICE

systemctl daemon-reload
systemctl enable jarvis
systemctl start jarvis
EOF


  tags = {
    Name = "jarvis-ec2"
  }
}
```

**variables.tf**
```
variable "aws_region" {
  default = "ap-south-1"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "ami_id" {
  default = "ami-02b8269d5e85954ef"
}

variable "key_name" {
  default = "add key pair"
}

variable "my_ip" {
  default = "your ip"
}

variable "jarvis_repo_url" {
  default = "https://github.com/iam-ganeshjadhav/Jarvis-Desktop-Voice-Assistant-Project.git"
}
```

To deploy:

```bash
terraform init
terraform apply -auto-approve
```

## Jenkins CI/CD Pipeline

2️⃣ **Jenkins CI/CD Pipeline**

Whenever you push to GitHub:

- GitHub → triggers Jenkins webhook
- Jenkins rsyncs updated code to EC2
- Jenkins restarts Jarvis
- EC2 instantly runs the new version
- Pipeline defined in `Jenkinsfile`


**jenkinsfile**
```
pipeline {
  agent any

  environment {
    TARGET = "ubuntu@13.126.157.57"
  }

  triggers {
    githubPush()
  }

  stages {
    stage("Checkout") {
      steps {
        git url: "https://github.com/iam-ganeshjadhav/Jarvis-Desktop-Voice-Assistant-Project.git", branch: "main"
      }
    }

    stage("Deploy to EC2") {
      steps {
        sshagent(['node-key']) {
          sh '''
          ssh -o StrictHostKeyChecking=no ubuntu@13.126.157.57 "echo Connected"
          rsync -avz --delete --exclude venv . ${TARGET}:/opt/jarvis
          ssh ${TARGET} "sudo systemctl restart jarvis"
          '''
        }
      }
    }
  }
}
```


## 3️⃣ Systemd Service

Jarvis runs as a Linux service:
```
sudo systemctl status jarvis
sudo systemctl restart jarvis
sudo journalctl -u jarvis -f
```
Runs automatically on every reboot.

## 🖼️ Screenshots

| Screenshot | Description |
|-----------|-------------|
| ![Jarvis Architecture](images/architecture.png) | Jarvis Architecture |
| ![Terraform Apply](images/terraform.png) | Terraform Apply |
| ![Jenkins Pipeline](images/jenkins.png) | Jenkins Pipeline |
| ![EC2 Deployment](images/ec2.png) | EC2 Deployment |

## 📚 What I Learned (Key Skills)

**🔹 Cloud (AWS)**
- EC2 provisioning
- IAM roles
- Security Groups
- Automated deployments

**🔹 DevOps**
- Jenkins Pipelines
- GitHub Webhooks
- SSH deployment (rsync)
- CI/CD automation
- Linux systemd services

**🔹 Terraform**
- EC2 creation
- User data scripting
- VPC/Subnets
- IAM instance profiles

**🔹 Python**
- Speech recognition
- Text-to-speech
- Working with modules
- Error handling
- Server-mode adaptation

## 👨‍💻 Author

- **Ganesh Jadhav**  
- Cloud & DevOps Enthusiast
- GitHub: https://github.com/iam-ganeshjadhav
- Gmail : jadhavg9370@gmail.com
