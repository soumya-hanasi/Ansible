# Ansible Nginx Installation and Configuration Project

## Project Overview

This project demonstrates how to automate the installation and configuration of an Nginx web server using Ansible.

The playbook follows Infrastructure as Code (IaC) principles and uses:

* Variables
* Playbooks
* Tasks
* Handlers
* Jinja2 Templates
* Service Management
* Verification Tasks
* Idempotent Operations

The automation installs Nginx, deploys a custom web page, configures the web server, starts the service, enables it on boot, and verifies successful deployment.

---

# Project Objectives

Automate the following tasks:

| Task                     | Description                           |
| ------------------------ | ------------------------------------- |
| Create Deployment User   | Create a dedicated deployment account |
| Update Package Cache     | Refresh package repositories          |
| Install Nginx            | Install web server package            |
| Create Website Directory | Prepare web content location          |
| Deploy Website           | Copy custom HTML page                 |
| Configure Nginx          | Deploy Jinja2 template                |
| Start Service            | Start Nginx                           |
| Enable Service           | Auto-start after reboot               |
| Verify Service           | Confirm service status                |
| Restart Service          | Trigger handler when changes occur    |

---

# Technologies Used

* Ansible
* Linux
* Nginx
* YAML
* Jinja2 Templates
* SSH
* Systemd

---

# Project Structure

```text
nginx-project/

├── inventory.ini
├── site.yml

├── files/
│   └── index.html

├── templates/
│   └── nginx.conf.j2

└── README.md
```

---

# Inventory Configuration

## inventory.ini

```ini
[webservers]
192.168.1.100

[webservers:vars]
ansible_user=ubuntu
```

Purpose:

Defines target servers where the playbook will run.

Verify connectivity:

```bash
ansible webservers -i inventory.ini -m ping
```

Expected Output:

```text
SUCCESS => {
  "ping": "pong"
}
```

---

# Main Playbook

## site.yml

```yaml
---
- name: Configure Nginx Web Server
  hosts: webservers
  become: yes

  vars:
    web_package: nginx
    web_service: nginx
    deploy_user: deploy
    website_root: /var/www/html
    server_name: localhost

  tasks:

    - name: Create deployment user
      user:
        name: "{{ deploy_user }}"
        shell: /bin/bash
        create_home: yes
        state: present

    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install nginx
      apt:
        name: "{{ web_package }}"
        state: present

    - name: Create website directory
      file:
        path: "{{ website_root }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Deploy website page
      copy:
        src: files/index.html
        dest: "{{ website_root }}/index.html"
      notify:
        - restart nginx

    - name: Deploy nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify:
        - restart nginx

    - name: Start nginx service
      service:
        name: "{{ web_service }}"
        state: started

    - name: Enable nginx service
      service:
        name: "{{ web_service }}"
        enabled: yes

    - name: Verify nginx is active
      command: systemctl is-active nginx
      register: nginx_status
      changed_when: false

    - name: Display nginx status
      debug:
        msg: "Nginx Status: {{ nginx_status.stdout }}"

  handlers:

    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

---

# Step-by-Step Execution Flow

```text
Connect to Server
        ↓
Become Root User
        ↓
Create Deployment User
        ↓
Update Package Cache
        ↓
Install Nginx
        ↓
Create Website Directory
        ↓
Deploy HTML Page
        ↓
Deploy Nginx Configuration
        ↓
Start Nginx Service
        ↓
Enable Service at Boot
        ↓
Verify Service Status
        ↓
Display Results
        ↓
Run Handler if Changes Detected
```

---

# Variables

```yaml
vars:
  web_package: nginx
  web_service: nginx
  deploy_user: deploy
  website_root: /var/www/html
  server_name: localhost
```

Purpose:

Stores reusable values.

Benefits:

* Easy maintenance
* Avoid hardcoding
* Better readability

Example:

```yaml
name: "{{ web_package }}"
```

Resolves to:

```yaml
name: nginx
```

---

# Task 1: Create Deployment User

```yaml
user:
  name: "{{ deploy_user }}"
```

Purpose:

Creates a dedicated deployment user.

Equivalent Command:

```bash
sudo useradd -m deploy
```

Benefits:

* Better security
* Easier access management

---

# Task 2: Update Package Cache

```yaml
apt:
  update_cache: yes
```

Purpose:

Refresh package repository metadata.

Equivalent Command:

```bash
sudo apt update
```

---

# Task 3: Install Nginx

```yaml
apt:
  name: nginx
  state: present
```

Purpose:

Installs Nginx if not already present.

Equivalent Command:

```bash
sudo apt install nginx -y
```

Idempotent Behavior:

* Installs once
* Skips if already installed

---

# Task 4: Create Website Directory

```yaml
file:
  path: /var/www/html
  state: directory
```

Purpose:

Creates website root directory.

Equivalent Commands:

```bash
sudo mkdir -p /var/www/html
sudo chmod 755 /var/www/html
```

---

# Task 5: Deploy Website Content

```yaml
copy:
  src: files/index.html
  dest: /var/www/html/index.html
```

Purpose:

Copies HTML page from control node to managed node.

Result:

Website becomes accessible through Nginx.

---

# Task 6: Deploy Nginx Configuration

```yaml
template:
  src: templates/nginx.conf.j2
  dest: /etc/nginx/sites-available/default
```

Purpose:

Deploys dynamic configuration using Jinja2.

Example Template:

```jinja2
server {
    listen 80;
    server_name {{ server_name }};
}
```

Rendered Output:

```nginx
server {
    listen 80;
    server_name localhost;
}
```

Benefits:

* Dynamic configuration
* Environment-specific customization

---

# Task 7: Start Nginx Service

```yaml
service:
  name: nginx
  state: started
```

Purpose:

Starts Nginx service.

Equivalent Command:

```bash
sudo systemctl start nginx
```

---

# Task 8: Enable Nginx Service

```yaml
service:
  name: nginx
  enabled: yes
```

Purpose:

Starts Nginx automatically after reboot.

Equivalent Command:

```bash
sudo systemctl enable nginx
```

---

# Task 9: Verify Service Status

```yaml
command: systemctl is-active nginx
```

Purpose:

Checks if Nginx is running.

Output:

```text
active
```

Stored in:

```yaml
nginx_status
```

---

# Task 10: Display Status

```yaml
debug:
  msg: "Nginx Status: {{ nginx_status.stdout }}"
```

Example Output:

```text
TASK [Display nginx status]

ok: [server] =>
{
  "msg": "Nginx Status: active"
}
```

---

# Handlers

## restart nginx

```yaml
handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

Purpose:

Restarts Nginx only when:

* Website content changes
* Configuration changes

Triggered by:

```yaml
notify:
  - restart nginx
```

Benefits:

* Avoids unnecessary restarts
* Minimizes downtime
* Production best practice

---

# Verification

Check service:

```bash
systemctl status nginx
```

Check website:

```bash
curl localhost
```

Expected Output:

```html
<h1>Nginx Installed Through Ansible</h1>
```

Or access:

```text
http://SERVER_IP
```

from a browser.

---

# Execution Commands

## Syntax Check

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
```

## Dry Run

```bash
ansible-playbook -i inventory.ini site.yml --check
```

## Execute Playbook

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# Expected End State

After successful execution:

✓ Deployment user created

✓ Package cache updated

✓ Nginx installed

✓ Website directory created

✓ Custom webpage deployed

✓ Nginx configuration applied

✓ Nginx service running

✓ Nginx enabled at boot

✓ Service verification completed

✓ Automatic restart on configuration changes

---

# Ansible Concepts Demonstrated

| Concept            | Implementation               |
| ------------------ | ---------------------------- |
| Inventory          | inventory.ini                |
| Playbook           | site.yml                     |
| Variables          | vars section                 |
| Tasks              | Installation & configuration |
| Handlers           | restart nginx                |
| Templates          | nginx.conf.j2                |
| Service Management | systemd                      |
| Verification       | status checks                |
| Idempotency        | Repeatable execution         |

---

# Learning Outcomes

After completing this project, you will understand:

* Ansible Playbooks
* Variables
* User Management
* Package Management
* File Management
* Jinja2 Templates
* Handlers
* Service Management
* Verification Tasks
* Idempotent Automation
* Infrastructure as Code

---

# Future Enhancements

* Convert playbook into Ansible Roles
* Add Ansible Galaxy support
* Configure SSL/TLS
* Deploy Multiple Virtual Hosts
* Integrate Monitoring
* Add CI/CD Pipeline
* Support Multiple Environments (Dev, QA, Prod)

---

# Author

Project: Nginx Installation and Configuration using Ansible

Skills Demonstrated:

* Ansible
* Linux Administration
* Nginx
* YAML
* Jinja2
* Infrastructure Automation
* DevOps Best Practices
