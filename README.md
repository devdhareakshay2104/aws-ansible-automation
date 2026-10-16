AWS Ansible Automation — EC2 Web Server Deployment

1. Project Overview

This project demonstrates how to use Ansible to automate the configuration of an Ubuntu web server running on AWS EC2.

Ansible runs on a controller instance called ansible-master and connects to a managed instance called ansible-node through SSH. It installs Nginx, deploys a custom HTML webpage, manages services, and verifies the deployment.

All setup instructions and code examples are documented in this README.

2. Project Objectives

Create Ubuntu EC2 instances for Ansible automation.

Install and configure Ansible on the controller.

Configure SSH authentication between the controller and managed node.

Manage the target server through an Ansible inventory.

Use playbooks, variables, loops, conditions, facts, handlers, templates, and roles.

Automate package, file, user, group, service, and cron management.

Learn error handling and Ansible Vault.

Deploy and verify an Nginx webpage automatically.

3. Technologies Used

Amazon Web Services (AWS)

Amazon EC2

Ubuntu Linux

Ansible

SSH

YAML

Jinja2 templates

Nginx

Git and GitHub

4. Architecture

                 Administrator
                       |
                       v
                Ansible Master
                 Ubuntu EC2
                       |
                 SSH / TCP 22
                       |
                       v
                 Ansible Node
                  Ubuntu EC2
                       |
                       v
                     Nginx
                       |
                       v
               Custom HTML Website
                  HTTP / TCP 80

The master executes Ansible commands and playbooks. The managed node receives tasks over SSH and runs them remotely.

5. Prerequisites

An AWS account.

Two Ubuntu EC2 instances.

SSH access to the controller.

An SSH key pair for initial access.

Ansible installed on the controller.

Basic knowledge of Linux commands and YAML.

Security: Keep SSH port 22 restricted to your own IP address where possible. Open HTTP port 80 to the public only when you want the demo website to be publicly accessible. Never upload private keys or AWS credentials.

6. Create EC2 Instances

Open AWS Console → EC2 → Instances → Launch instances.

Create these instances:

Setting

Ansible Master

Ansible Node

Name

ansible-master

ansible-node

Operating system

Ubuntu Server

Ubuntu Server

Instance type

Suitable free-tier-eligible type, if available

Suitable free-tier-eligible type, if available

Network

Same VPC

Same VPC

SSH key

Your selected key pair

Your selected key pair

SSH access

Your IP address

Preferably master access for private SSH

Use a suitable instance type based on your account's current free-tier eligibility and pricing.

Both instances should be in a network where the master can reach the node over SSH. For this lab, the easiest configuration is to place them in the same VPC and allow SSH from the master's security group to the node.

If you want to install packages from Ubuntu repositories, the node needs outbound internet connectivity or another configured package source.

7. Connect to the Ansible Master

From your terminal, connect using the public IPv4 address of the master.

ssh -i /path/to/your-key.pem ubuntu@MASTER_PUBLIC_IP

Replace /path/to/your-key.pem and MASTER_PUBLIC_IP with your actual local key path and master public IP.

Do not paste the contents of your private key into this README or GitHub.

8. Install Ansible on the Master

Update the package cache:

sudo apt update

Install Ansible:

sudo apt install ansible -y

Verify the installation:

ansible --version

Create the project directory:

mkdir -p ~/ansible-project
cd ~/ansible-project

ed how Ansible can execute administrative tasks remotely without requiring manual login to the managed node.
