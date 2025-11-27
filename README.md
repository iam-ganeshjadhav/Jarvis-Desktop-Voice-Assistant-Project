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
