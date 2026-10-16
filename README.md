<div align="center">

🚀 AWS Ansible Automation
Automate Ubuntu EC2 Configuration & Nginx Web Server Deployment
<p>
  <img src="https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws&logoColor=white" alt="AWS EC2">
  <img src="https://img.shields.io/badge/Automation-Ansible-red?logo=ansible&logoColor=white" alt="Ansible">
  <img src="https://img.shields.io/badge/OS-Ubuntu-E95420?logo=ubuntu&logoColor=white" alt="Ubuntu">
  <img src="https://img.shields.io/badge/Web_Server-Nginx-009639?logo=nginx&logoColor=white" alt="Nginx">
  <img src="https://img.shields.io/badge/Documentation-README.md-blue?logo=markdown&logoColor=white" alt="README">
</p>

A hands-on DevOps project demonstrating server automation with Ansible on AWS EC2.
</div>

📌 Table of Contents
- Overview
- Project Goals
- Architecture
- Technology Stack
- Prerequisites
- Step 1 — Create EC2 Instances
- Step 2 — Connect to the Master
- Step 3 — Install Ansible
- Step 4 — Configure SSH
- Step 5 — Configure Inventory
- Step 6 — Test Connectivity
- Step 7 — Install Nginx
- Ansible Concepts Practiced
- Final Deployment Project
- Troubleshooting
- Security and Cost Notes
- Skills Demonstrated
- Conclusion
📖 Overview
This project uses Ansible to automate configuration of an Ubuntu web server hosted on Amazon EC2.
Ansible runs from a controller machine named ansible-master and connects to a managed machine named ansible-node over SSH. It can install Nginx, deploy a custom webpage, manage services, and verify the HTTP response.
Repository format: This repository is intentionally documented in a single README.md file. Commands and code samples are shown below; save the relevant code blocks into files on the EC2 controller when you want to execute them.

🎯 Project Goals
- Provision two Ubuntu EC2 instances for the lab.
- Install Ansible on the controller node.
- Configure SSH authentication to the managed node.
- Learn inventories, ad-hoc commands, and playbooks.
- Practice variables, loops, conditions, facts, handlers, templates, and roles.
- Automate packages, files, users, groups, services, and cron jobs.
- Understand error handling and Ansible Vault.
- Deploy and verify an Nginx webpage.
🏗️ Architecture
                    👤 Administrator
                           |
                           v
                 ┌────────────────────┐
                 │   Ansible Master   │
                 │     Ubuntu EC2     │
                 └────────────────────┘
                           |
                     SSH / TCP 22
                           |
                           v
                 ┌────────────────────┐
                 │    Ansible Node    │
                 │     Ubuntu EC2     │
                 └────────────────────┘
                           |
                           v
                 ┌────────────────────┐
                 │       Nginx        │
                 │  Custom HTML Page  │
                 └────────────────────┘
                           |
                     HTTP / TCP 80
                           |
                           v
                     Web Browser
How it works: The master reads the inventory and playbook, connects to the node over SSH, and executes tasks remotely. Nginx serves the deployed webpage.
🧰 Technology Stack
Technology	Purpose
AWS EC2	Host the controller and managed server
Ubuntu Linux	Operating system
Ansible	Configuration management and automation
SSH	Secure remote connection
YAML	Playbook and configuration syntax
Jinja2	Dynamic templates
Nginx	Web server
Git and GitHub	Version control and project documentation


✅ Prerequisites
- An AWS account.
- Two Ubuntu EC2 instances.
- SSH access to the master.
- An SSH key pair for initial EC2 access.
- Ansible installed on the master.
- Basic Linux and YAML knowledge.
🔐 Security: Keep SSH port 22 restricted to trusted sources. Allow HTTP port 80 publicly only when you want to demonstrate the website. Never commit private keys, .pem files, AWS credentials, or real passwords.

Step 1 — Create EC2 Instances
In the AWS Console, open EC2 → Instances → Launch instances.
Create these two instances:
Setting	Ansible Master	Ansible Node
Name	ansible-master	ansible-node
AMI	Ubuntu Server	Ubuntu Server
VPC	Same VPC	Same VPC
SSH key	Your selected key pair	Your selected key pair
SSH access	Your IP address	Preferably allow SSH from the master's security group


Choose an instance type appropriate for your account and current AWS pricing. Free-tier eligibility depends on account status and the instance type.
Ensure the master can reach the node on TCP port 22. The node also needs outbound access to Ubuntu package repositories, or another configured package source, to install packages.
Step 2 — Connect to the Master
From your local terminal, connect using the master's current public IPv4 address:
ssh -i /path/to/your-key.pem ubuntu@MASTER_PUBLIC_IP
Replace the placeholders with your local private-key path and the master's actual public IP.
Do not share or upload the private key.
Step 3 — Install Ansible
On the master:
sudo apt update
sudo apt install ansible -y
ansible --version
Create a workspace:
mkdir -p ~/ansible-project
cd ~/ansible-project
Step 4 — Configure SSH Authentication
Generate an SSH key on the master if you do not already have one:
ssh-keygen -t ed25519 -C "ansible-master"
Accept the default location if appropriate. Do not overwrite an existing key unless you intend to replace it.
Display the public key:
cat ~/.ssh/id_ed25519.pub
Add that public key to the managed node's ~/.ssh/authorized_keys using a secure method. If password-based SSH is enabled on the node, you can use:
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@NODE_PRIVATE_IP
Test the connection:
ssh -i ~/.ssh/id_ed25519 ubuntu@NODE_PRIVATE_IP
After verifying access, return to the master:
exit
Step 5 — Configure the Ansible Inventory
The inventory tells Ansible which machines to manage and how to connect to them.
Open the inventory file:
sudo nano /etc/ansible/hosts
Add this entry, replacing NODE_PRIVATE_IP with the current private IP of your node:
[web]
NODE_PRIVATE_IP ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
Save and exit the editor.
Step 6 — Test Connectivity and Run Ad-Hoc Commands
Test the Ansible connection:
ansible web -m ping
Expected result:
SUCCESS => {
    "changed": false,
    "ping": "pong"
}
Try a few ad-hoc commands:
ansible web -m command -a "uptime"
ansible web -m command -a "date"
ansible web -m command -a "free -h"
ansible web -m command -a "df -h"
Check Nginx status with elevated privileges:
ansible web -b -m command -a "systemctl is-active nginx"
