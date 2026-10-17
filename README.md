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
  
How it works: 

The master reads the inventory and playbook, connects to the node over SSH, and executes tasks remotely. Nginx serves the deployed webpage.

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


🔐 Security: 

Keep SSH port 22 restricted to trusted sources. Allow HTTP port 80 publicly only when you want to demonstrate the website. Never commit private keys, .pem files, AWS credentials, or real passwords.


Step 1 — Create EC2 Instances:

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


Step 2 — Connect to the Master:

From your local terminal, connect using the master's current public IPv4 address:
ssh -i /path/to/your-key.pem ubuntu@MASTER_PUBLIC_IP
Replace the placeholders with your local private-key path and the master's actual public IP.
Do not share or upload the private key.


Step 3 — Install Ansible On the master:

sudo apt update
sudo apt install ansible -y
ansible --version

Create a workspace:

mkdir -p ~/ansible-project
cd ~/ansible-project


Step 4 — Configure SSH Authentication:

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


Step 5 — Configure the Ansible Inventory:

The inventory tells Ansible which machines to manage and how to connect to them.
Open the inventory file:

sudo nano /etc/ansible/hosts
Add this entry, replacing NODE_PRIVATE_IP with the current private IP of your node:

[web]
NODE_PRIVATE_IP ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519

Save and exit the editor.


Step 6 — Test Connectivity and Run Ad-Hoc Commands:

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


Step 7 — Install Nginx with a Playbook:


Create a playbook on the master:

nano nginx-playbook.yml

Add:

---
- name: Install and configure Nginx
  hosts: web
  become: yes

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
      
Run the playbook:

ansible-playbook nginx-playbook.yml

Verify the service and local HTTP response:

ansible web -b -m command -a "systemctl is-active nginx"
ansible web -m command -a "curl -I http://127.0.0.1"
ansible web -b -m command -a "systemctl is-active nginx"


🧠 Ansible Concepts Practiced:

The following examples are documentation snippets. To execute one, save it in a file with the indicated name on the controller and run the corresponding command.

<details>
<summary><strong>1. Variables</strong></summary>

Variables allow values to be reused.

Save as variables.yml:

---
- name: Demonstrate variables
  hosts: web
  become: yes

  vars:
    package_name: nginx
    service_name: nginx

  tasks:
    - name: Install package
      apt:
        name: "{{ package_name }}"
        state: present

    - name: Start service
      service:
        name: "{{ service_name }}"
        state: started
        enabled: yes

      
Run:

ansible-playbook variables.yml

</details>

<details>
<summary><strong>2. Loops</strong></summary>

Loops apply a task to multiple items.
Save as loops.yml:
---
- name: Install multiple packages
  hosts: web
  become: yes

  vars:
    packages:
      - nginx
      - curl
      - unzip

  tasks:
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
        update_cache: yes
      loop: "{{ packages }}"
Run:
ansible-playbook loops.yml
</details>

<details>
<summary><strong>3. Conditions</strong></summary>

The when keyword runs a task only when its condition is true.
Save as conditions.yml:
---
- name: Demonstrate conditions
  hosts: web
  become: yes
  gather_facts: yes

  tasks:
    - name: Install Nginx on Ubuntu
      apt:
        name: nginx
        state: present
      when: ansible_distribution == "Ubuntu"
Run:
ansible-playbook conditions.yml
</details>

<details>
<summary><strong>4. Facts</strong></summary>

Facts provide system information such as hostname, operating system, IP address, and memory.
Save as facts.yml:
---
- name: Display server facts
  hosts: web
  gather_facts: yes

  tasks:
    - name: Display hostname
      debug:
        msg: "Hostname: {{ ansible_hostname }}"

    - name: Display operating system
      debug:
        msg: "OS: {{ ansible_distribution }}"

    - name: Display private IP
      debug:
        msg: "Private IP: {{ ansible_default_ipv4.address }}"

    - name: Display memory
      debug:
        msg: "Memory: {{ ansible_memtotal_mb }} MB"
Run:
ansible-playbook facts.yml
</details>

<details>
<summary><strong>5. Handlers</strong></summary>

A handler runs when notified by a task that reports a change.
Save as handlers.yml:
---
- name: Demonstrate handlers
  hosts: web
  become: yes

  tasks:
    - name: Deploy a webpage
      copy:
        content: "<h1>Deployed using Ansible</h1>"
        dest: /var/www/html/index.html
      notify: Reload Nginx

  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
Run:
ansible-playbook handlers.yml
</details>

<details>
<summary><strong>6. Jinja2 Templates</strong></summary>

Templates insert variables and gathered facts into files.
Example template:
<!DOCTYPE html>
<html>
<head>
    <title>{{ website_title }}</title>
</head>
<body>
    <h1>{{ website_title }}</h1>
    <p>{{ website_message }}</p>
    <p>Server: {{ ansible_hostname }}</p>
    <p>Operating System: {{ ansible_distribution }}</p>
</body>
</html>
The Ansible template module renders the Jinja2 expressions and places the result on the managed node.
</details>

<details>
<summary><strong>7. Roles</strong></summary>

Roles organize reusable tasks, variables, handlers, templates, and files.
Create a role:
cd ~/ansible-project
ansible-galaxy role init nginx-role
Example nginx-role/tasks/main.yml:
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
Example playbook:
---
- name: Run Nginx role
  hosts: web
  become: yes

  roles:
    - nginx-role
Run the playbook from a directory where Ansible can locate the role, or configure the role search path.
</details>

<details>
<summary><strong>8. File Management</strong></summary>

Example:
---
- name: Manage files
  hosts: web
  become: yes

  tasks:
    - name: Create directory
      file:
        path: /opt/ansible-demo
        state: directory
        mode: '0755'

    - name: Create a file with content
      copy:
        content: "Created using Ansible\n"
        dest: /opt/ansible-demo/info.txt
        mode: '0644'
</details>

<details>
<summary><strong>9. Users and Groups</strong></summary>

Example:
---
- name: Manage Linux users and groups
  hosts: web
  become: yes

  tasks:
    - name: Create group
      group:
        name: devops
        state: present

    - name: Create user
      user:
        name: devopsuser
        group: devops
        shell: /bin/bash
        create_home: yes
        state: present
</details>

<details>
<summary><strong>10. Package Management</strong></summary>

Example:
---
- name: Manage packages
  hosts: web
  become: yes

  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install required packages
      apt:
        name:
          - nginx
          - curl
          - unzip
          - tree
        state: present
</details>

<details>
<summary><strong>11. Cron Jobs</strong></summary>

Example: write the date to a log file every minute.
---
- name: Configure cron
  hosts: web
  become: yes

  tasks:
    - name: Record the date every minute
      cron:
        name: "Record date and time"
        minute: "*"
        job: "/usr/bin/date >> /var/log/ansible-cron.log"
        state: present
</details>

<details>
<summary><strong>12. Service Management</strong></summary>

Example:
---
- name: Manage Nginx
  hosts: web
  become: yes

  tasks:
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
</details>

<details>
<summary><strong>13. Copy and Fetch Files</strong></summary>

Copy from controller to node:
- name: Copy file to node
  copy:
    src: sample.txt
    dest: /tmp/sample.txt
    mode: '0644'
Fetch from node to controller:
- name: Fetch file from node
  fetch:
    src: /tmp/sample.txt
    dest: ./fetched-files/
    flat: yes
</details>

<details>
<summary><strong>14. Error Handling</strong></summary>

Use block, rescue, and always to handle task failures.
---
- name: Demonstrate error handling
  hosts: web

  tasks:
    - name: Run tasks with error handling
      block:
        - name: Run a successful command
          command: /bin/true

        - name: Demonstrate a failure
          command: /bin/false

      rescue:
        - name: Handle the failure
          debug:
            msg: "A task failed; rescue is running."

      always:
        - name: Always execute
          debug:
            msg: "The always section has executed."
</details>

<details>
<summary><strong>15. Ansible Vault</strong></summary>

Ansible Vault encrypts files containing sensitive data.
Example practice file:
---
db_username: admin
db_password: DemoPass123!
Save as secrets.yml and encrypt it:
ansible-vault encrypt secrets.yml
View or edit the encrypted file:
ansible-vault view secrets.yml
ansible-vault edit secrets.yml
The example password is for practice only. Never store real secrets in a public README or repository.
</details>

🚀 Final Deployment Project

This final playbook installs Nginx, deploys a dynamic webpage using a Jinja2 template, ensures Nginx is running, and verifies the HTTP response.
1. Project files on the controller
Create these files on the master for execution:

final-project/
├── site.yml
├── vars.yml
└── templates/
    └── index.html.j2
   
The files above are used on the EC2 controller. This GitHub repository itself remains README-only, as intended.

3. Variables — vars.yml
---
website_title: "My AWS Ansible Project"
website_message: "Deployed automatically using Ansible"
3. Template — templates/index.html.j2
<!DOCTYPE html>
<html>
<head>
    <title>{{ website_title }}</title>
</head>
<body>
    <h1>{{ website_title }}</h1>
    <p>{{ website_message }}</p>
    <p>Server: {{ ansible_hostname }}</p>
    <p>Operating System: {{ ansible_distribution }}</p>
</body>
</html>
4. Playbook — site.yml
---
- name: Deploy a web server on AWS EC2
  hosts: web
  become: yes
  gather_facts: yes

  vars_files:
    - vars.yml

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
        cache_valid_time: 3600

    - name: Deploy custom webpage
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'
      notify: Reload Nginx

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Verify webpage response
      uri:
        url: http://127.0.0.1
        status_code: 200
      register: website_check

    - name: Display deployment result
      debug:
        msg: "Website deployment successful. HTTP status: {{ website_check.status }}"

  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
5. Execute the deployment
cd ~/ansible-project/final-project
ansible-playbook -i /etc/ansible/hosts site.yml
Expected result: the playbook completes without failures and the HTTP check returns status 200.
6. Verify the webpage
Check the page from the master through the managed node:
ansible web -m command -a "curl -s http://127.0.0.1"
To view it externally, open the node's current public IPv4 address in a browser:
http://NODE_PUBLIC_IP
The node's security group must allow inbound HTTP on port 80 for public access.


🛠️ Troubleshooting

<details>
<summary><strong>Ansible cannot connect to the node</strong></summary>

Check the inventory:
cat /etc/ansible/hosts
Test connectivity:
ansible web -m ping
Confirm the private IP, SSH username, private-key path, and security-group rules. The managed node must allow SSH from the master.
</details>

<details>
<summary><strong>APT cache update fails</strong></summary>

Test outbound connectivity:
ansible web -m command -a "curl -I --connect-timeout 10 https://archive.ubuntu.com"
If the request times out, check outbound security-group rules, subnet routes, internet gateway or NAT configuration, network ACLs, and the operating system firewall.
</details>

<details>
<summary><strong>The webpage does not open</strong></summary>

Check Nginx:
ansible web -b -m command -a "systemctl is-active nginx"
Check the local HTTP response:
ansible web -m command -a "curl -I http://127.0.0.1"
Verify the node's current public IP and inbound HTTP security-group rule.
</details>

🔐 Security and Cost Management

- Never upload private SSH keys, .pem files, AWS credentials, or real passwords.
- Restrict SSH access to trusted sources.
- Expose HTTP publicly only when needed for the demonstration.
- Stop EC2 instances when you are not using the lab.
- Stopping instances generally ends compute charges, but EBS storage and other resources may still incur charges.
- Review AWS billing and clean up unused resources.
- 
💼 Skills Demonstrated

- AWS EC2 and Ubuntu administration
- SSH authentication and Ansible inventory
- Ansible ad-hoc commands and playbooks
- Variables, loops, conditions, and facts
- Handlers, templates, and roles
- Package, file, user, group, and service management
- Cron jobs, file transfer, and error handling
- Ansible Vault
- Nginx deployment and HTTP verification
- Git and GitHub documentation

🎓 Conclusion

This project demonstrates how Ansible automates repetitive server configuration tasks on AWS EC2. The final deployment combines package installation, Jinja2 templates, service management, and HTTP verification into a practical web-server automation workflow.
