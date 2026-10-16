Ansible Configuration Management on AWS EC2

📌 Project Overview

This project demonstrates how to configure and manage an Ubuntu EC2 instance using Ansible on AWS. It covers setting up an Ansible master and managed node, configuring passwordless SSH authentication, creating an inventory file, executing ad-hoc commands, and installing and running the Nginx web server remotely.

🎯 Project Objectives

Create an Ansible master and managed node on AWS EC2.

Install and configure Ansible on Ubuntu.

Establish SSH connectivity between the master and node.

Configure an Ansible inventory.

Test connectivity using the Ansible ping module.

Execute Linux commands remotely using Ansible ad-hoc commands.

Install and start Nginx using Ansible.

Host and access the default Nginx welcome page through a public IP address.

🏗️ Architecture

                 Developer / User
                        |
                        | HTTP (Port 80)
                        v
               +-------------------+
               |   Web Browser     |
               +-------------------+
                        |
                        v
        +--------------------------------+
        |          AWS Cloud             |
        |       Mumbai Region            |
        |         ap-south-1              |
        |                                |
        |  +--------------------------+  |
        |  |   ansible-master         |  |
        |  |   Ubuntu EC2              |  |
        |  |                           |  |
        |  |   Ansible                 |  |
        |  |   SSH Private Key         |  |
        |  |   Inventory File          |  |
        |  +-------------+-------------+  |
        |                |                |
        |                | SSH (Port 22)  |
        |                |                |
        |                v                |
        |  +--------------------------+  |
        |  |   ansible-node           |  |
        |  |   Ubuntu EC2              |  |
        |  |                           |  |
        |  |   SSH Public Key          |  |
        |  |   Nginx Web Server        |  |
        |  |   Port 80                 |  |
        |  +--------------------------+  |
        |                                |
        |  VPC + Public Subnet            |
        |  Internet Gateway               |
        +--------------------------------+

Architecture Components

Component

Purpose

AWS EC2

Provides virtual machines for the project.

Ansible Master

Executes Ansible commands and manages the node.

Ansible Node

Managed server where tasks are executed.

SSH

Provides secure communication between the master and node.

Inventory

Stores the managed node's IP address and connection details.

Nginx

Web server installed on the managed node.

VPC

Provides an isolated AWS network for the infrastructure.

Internet Gateway

Enables internet connectivity for resources in a properly configured public subnet.

Security Groups

Control inbound and outbound instance traffic.

🛠️ Technologies Used

Cloud Platform: Amazon Web Services (AWS)

Compute: Amazon EC2

Operating System: Ubuntu Linux

Configuration Management: Ansible

Remote Access: SSH

Web Server: Nginx

Networking: VPC, Subnets, Internet Gateway, Route Tables, Security Groups

Web Protocol: HTTP

⚙️ Implementation Steps

Step 1: Create EC2 Instances

Created two Ubuntu EC2 instances in the AWS Mumbai region:

ansible-master — Ansible controller.

ansible-node — Managed server.

Both instances were configured with suitable networking and security-group rules.

Step 2: Connect to the Master

Connected to the ansible-master instance using EC2 Instance Connect.

Verified the username and hostname:

whoami
hostname

Step 3: Install Ansible

Updated the package list:

sudo apt update

Installed Ansible:

sudo apt install ansible -y

Verified the installation:

ansible --version

Step 4: Configure SSH Authentication

Generated an Ed25519 SSH key pair on the master:

ssh-keygen -t ed25519

Displayed the public key:

cat ~/.ssh/id_ed25519.pub

Added the master's public key to the managed node's ~/.ssh/authorized_keys file and configured the appropriate SSH permissions.

Tested the SSH connection from the master:

ssh -i ~/.ssh/id_ed25519 ubuntu@NODE_PRIVATE_IP

Replaced NODE_PRIVATE_IP with the managed node's private IP address.

Step 5: Configure the Ansible Inventory

Created the inventory directory and file:

sudo mkdir -p /etc/ansible
sudo nano /etc/ansible/hosts

Added the managed node configuration:

[web]
NODE_PRIVATE_IP ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519

Replaced NODE_PRIVATE_IP with the current private IP address of ansible-node.

Verified the inventory:

ansible-inventory --list

Step 6: Test Ansible Connectivity

Executed the Ansible ping module:

ansible web -m ping

Expected result:

SUCCESS
"changed": false,
"ping": "pong"

This confirmed that the master could communicate with the managed node through Ansible.

Step 7: Execute Ad-Hoc Commands

Executed Linux commands remotely on the managed node.

Check system uptime

ansible web -m command -a "uptime"

Display the current date

ansible web -m command -a "date"

Check memory usage

ansible web -m command -a "free -h"

Check disk usage

ansible web -m command -a "df -h"

These commands demonstrated how Ansible can execute administrative tasks remotely without requiring manual login to the managed node.
