# AWS Ansible Automation — EC2 Web Server Deployment

## 1. Project Overview :

This project demonstrates how to use Ansible to automate the configuration of an Ubuntu web server running on AWS EC2.

Ansible runs on a controller instance called `ansible-master` and connects to a managed instance called `ansible-node` through SSH. It installs Nginx, deploys a custom HTML webpage, manages services, and verifies the deployment.

All setup instructions and code examples are documented in this README.

## 2. Project Objectives :

- Create Ubuntu EC2 instances for Ansible automation.
- Install and configure Ansible on the controller.
- Configure SSH authentication between the controller and managed node.
- Manage the target server through an Ansible inventory.
- Use playbooks, variables, loops, conditions, facts, handlers, templates, and roles.
- Automate package, file, user, group, service, and cron management.
- Learn error handling and Ansible Vault.
- Deploy and verify an Nginx webpage automatically.

## 3. Technologies Used :

- Amazon Web Services (AWS)
- Amazon EC2
- Ubuntu Linux
- Ansible
- SSH
- YAML
- Jinja2 templates
- Nginx
- Git and GitHub

## 4. Architecture :

```text
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
```

The master executes Ansible commands and playbooks. The managed node receives tasks over SSH and runs them remotely.

## 5. Prerequisites :

- An AWS account.
- Two Ubuntu EC2 instances.
- SSH access to the controller.
- An SSH key pair for initial access.
- Ansible installed on the controller.
- Basic knowledge of Linux commands and YAML.

**Security:** Keep SSH port 22 restricted to your own IP address where possible. Open HTTP port 80 to the public only when you want the demo website to be publicly accessible. Never upload private keys or AWS credentials.

## 6. Create EC2 Instances

Open AWS Console → EC2 → Instances → Launch instances.

Create these instances:

| Setting | Ansible Master | Ansible Node |
|---|---|---|
| Name | ansible-master | ansible-node |
| Operating system | Ubuntu Server | Ubuntu Server |
| Instance type | Suitable free-tier-eligible type, if available | Suitable free-tier-eligible type, if available |
| Network | Same VPC | Same VPC |
| SSH key | Your selected key pair | Your selected key pair |
| SSH access | Your IP address | Preferably master access for private SSH |

Use a suitable instance type based on your account's current free-tier eligibility and pricing.

Both instances should be in a network where the master can reach the node over SSH. For this lab, the easiest configuration is to place them in the same VPC and allow SSH from the master's security group to the node.

If you want to install packages from Ubuntu repositories, the node needs outbound internet connectivity or another configured package source.

## 7. Connect to the Ansible Master

From your terminal, connect using the public IPv4 address of the master.

```bash
ssh -i /path/to/your-key.pem ubuntu@MASTER_PUBLIC_IP
```

Replace `/path/to/your-key.pem` and `MASTER_PUBLIC_IP` with your actual local key path and master public IP.

Do not paste the contents of your private key into this README or GitHub.

## 8. Install Ansible on the Master

Update the package cache:

```bash
sudo apt update
```

Install Ansible:

```bash
sudo apt install ansible -y
```

Verify the installation:

```bash
ansible --version
```

Create the project directory:

```bash
mkdir -p ~/ansible-project
cd ~/ansible-project
```

## 9. Configure SSH Authentication

Ansible normally uses SSH to connect to Linux managed nodes.

Generate an SSH key on the master:

```bash
ssh-keygen -t ed25519 -C "ansible-master"
```

Accept the default path for the practice lab. Do not overwrite an existing key unless you intend to replace it.

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the public key and add it to the managed node's `~/.ssh/authorized_keys` file using a secure method. Alternatively, if password-based SSH is enabled on the node, you can use `ssh-copy-id`:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@NODE_PRIVATE_IP
```

Replace `NODE_PRIVATE_IP` with the node's actual private IPv4 address.

Test SSH connectivity:

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@NODE_PRIVATE_IP
```

Once the connection works, exit the node's shell:

```bash
exit
```

## 10. Configure the Ansible Inventory

An inventory tells Ansible which managed servers to contact and how to connect to them.

Open the inventory:

```bash
sudo nano /etc/ansible/hosts
```

Add:

```ini
[web]
NODE_PRIVATE_IP ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

Replace `NODE_PRIVATE_IP` with the current private IP of `ansible-node`.

Save and exit the editor.

Test connectivity:

```bash
ansible web -m ping
```

Expected result:

```text
SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

The IP address in the output will be your actual managed node address.

## 11. Run Ad-Hoc Commands

Ansible ad-hoc commands execute one-off tasks without requiring a playbook.

Check uptime:

```bash
ansible web -m command -a "uptime"
```

Check the date:

```bash
ansible web -m command -a "date"
```

Check memory:

```bash
ansible web -m command -a "free -h"
```

Check disk usage:

```bash
ansible web -m command -a "df -h"
```

Run commands with elevated privileges when required:

```bash
ansible web -b -m command -a "systemctl is-active nginx"
```

## 12. Install Nginx with Ansible

Create a playbook:

```bash
nano nginx-playbook.yml
```

Code:

```yaml
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
```

Run:

```bash
ansible-playbook nginx-playbook.yml
```

Verify the service:

```bash
ansible web -b -m command -a "systemctl is-active nginx"
```

Verify the local HTTP response on the node:

```bash
ansible web -m command -a "curl -I http://127.0.0.1"
```

## 13. Ansible Variables

Variables allow reusable values to be defined once and referenced throughout a playbook.

Example:

```yaml
---
- name: Demonstrate Ansible variables
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
```

Run:

```bash
ansible-playbook variables.yml
```

Save the example as `variables.yml` on the controller if you want to execute it.

## 14. Ansible Loops

Loops allow the same task to process multiple items.

Example:

```yaml
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
```

Run:

```bash
ansible-playbook loops.yml
```

## 15. Ansible Conditions

The `when` keyword executes a task only when a condition is true.

Example:

```yaml
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
```

Run:

```bash
ansible-playbook conditions.yml
```

## 16. Ansible Facts

Facts are system details collected automatically from managed nodes.

Example:

```yaml
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
```

Run:

```bash
ansible-playbook facts.yml
```

## 17. Ansible Handlers

Handlers run when notified by a task that reports a change.

Example:

```yaml
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
```

Run:

```bash
ansible-playbook handlers.yml
```

The handler runs only when the copy task changes the file.

## 18. Ansible Templates

Jinja2 templates allow variables and gathered facts to be inserted into files.

Example template content:

```html
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
```

The template module processes the expressions and writes the rendered file to the managed node.

## 19. Ansible Roles

Roles organize tasks, variables, handlers, templates, and other files into a reusable directory structure.

Create a role:

```bash
cd ~/ansible-project
ansible-galaxy role init nginx-role
```

Example task file for `nginx-role/tasks/main.yml`:

```yaml
---
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
```

Example playbook:

```yaml
---
- name: Run Nginx role
  hosts: web
  become: yes

  roles:
    - nginx-role
```

For execution, save these examples in their respective files on the master and configure the role search path correctly.

## 20. Ansible File Management

Example:

```yaml
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
```

Run:

```bash
ansible-playbook file-management.yml
```

## 21. Ansible Users and Groups

Example:

```yaml
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
```

Run:

```bash
ansible-playbook users-groups.yml
```

## 22. Ansible Package Management

Example:

```yaml
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
```

Run:

```bash
ansible-playbook package-management.yml
```

## 23. Ansible Cron Jobs

Example:

```yaml
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
```

Run:

```bash
ansible-playbook cron-job.yml
```

## 24. Ansible Service Management

Example:

```yaml
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
```

Run:

```bash
ansible-playbook service-management.yml
```

## 25. Copy and Fetch Files

Copy a file from the controller to the managed node:

```yaml
- name: Copy file to node
  copy:
    src: sample.txt
    dest: /tmp/sample.txt
    mode: '0644'
```

Fetch a file from the managed node to the controller:

```yaml
- name: Fetch file from node
  fetch:
    src: /tmp/sample.txt
    dest: ./fetched-files/
    flat: yes
```

Run the tasks using an appropriate playbook.

## 26. Error Handling

Use `block`, `rescue`, and `always` to manage task failures.

Example:

```yaml
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
```

Run:

```bash
ansible-playbook error-handling.yml
```

## 27. Ansible Vault

Ansible Vault encrypts sensitive files.

Create a practice variables file:

```yaml
---
db_username: admin
db_password: DemoPass123!
```

Save this example as `secrets.yml` on the controller, then encrypt it:

```bash
ansible-vault encrypt secrets.yml
```

View the contents:

```bash
ansible-vault view secrets.yml
```

Edit the encrypted file:

```bash
ansible-vault edit secrets.yml
```

Never put real passwords, AWS credentials, or private keys in this README.

## 28. Final Project: Automated Nginx Deployment

The following is the final deployment playbook. It installs Nginx, deploys a dynamic webpage, ensures the service is running, and checks the HTTP response.

### Variables

```yaml
---
website_title: "My AWS Ansible Project"
website_message: "Deployed automatically using Ansible"
```

### Jinja2 template

```html
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
```

### Main playbook

```yaml
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
```

### Final project directory on the controller

For execution, create these files on the master:

```text
final-project/
├── site.yml
├── vars.yml
└── templates/
    └── index.html.j2
```

Save each corresponding code block in its named file. The directory structure above is for the EC2 execution environment; this GitHub repository documents the code in README.md only.

### Execute the final playbook

```bash
cd ~/ansible-project/final-project
ansible-playbook -i /etc/ansible/hosts site.yml
```

### Verify the website

Check the HTTP response from the node:

```bash
ansible web -m command -a "curl -s http://127.0.0.1"
```

To access the website externally, open the current public IPv4 address of the node in a browser:

```text
http://NODE_PUBLIC_IP
```

Ensure Nginx is running and the node's security group allows inbound HTTP on port 80.

## 29. Troubleshooting

### Ansible cannot connect to the node

Check the inventory:

```bash
cat /etc/ansible/hosts
```

Test connectivity:

```bash
ansible web -m ping
```

Confirm that the private IP is current, the SSH username is correct, the key path is correct, and the node's security group allows SSH from the master.

### APT cache update fails

Test outbound connectivity:

```bash
ansible web -m command -a "curl -I --connect-timeout 10 https://archive.ubuntu.com"
```

If the request times out, check the node's outbound security group rules, subnet route table, internet gateway or NAT configuration, network ACL, and operating system firewall.

### The webpage is not accessible

Check the service:

```bash
ansible web -b -m command -a "systemctl is-active nginx"
```

Check the HTTP response:

```bash
ansible web -m command -a "curl -I http://127.0.0.1"
```

Then verify the current public IP and inbound HTTP security group rule.

## 30. Security and Cost Management

- Do not upload private SSH keys, `.pem` files, AWS credentials, or real passwords.
- Keep SSH access restricted to trusted sources.
- Expose HTTP only when needed for the demo.
- Stop EC2 instances when the lab is not in use.
- Stopping instances generally ends compute charges, but EBS storage and other resources can still incur charges.
- Review AWS billing and clean up resources you no longer need.
- Keep the repository free of sensitive information.

## 31. Skills Demonstrated

- AWS EC2 and Linux administration
- SSH authentication
- Ansible inventory and ad-hoc commands
- YAML playbooks
- Variables, loops, conditions, and facts
- Handlers, templates, and roles
- Package and service management
- File and user management
- Cron jobs and error handling
- Ansible Vault
- Automated Nginx deployment
- Git and GitHub documentation

## 32. Conclusion

This project demonstrates how Ansible can automate repetitive server configuration tasks on AWS EC2. The final deployment provides a practical example of installing Nginx, deploying a dynamic webpage, and verifying the result through an Ansible playbook.
