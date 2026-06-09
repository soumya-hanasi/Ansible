# Ansible Nginx Deployment Project

## Main Playbook: site.yml

The playbook automates the complete setup and configuration of an Nginx web server.

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
        owner: www-data
        group: www-data
        mode: '0644'
      notify:
        - restart nginx

    - name: Deploy nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
        owner: root
        group: root
        mode: '0644'
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

When the playbook runs, Ansible executes tasks sequentially.

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
Deploy Website Files
        ↓
Deploy Nginx Configuration
        ↓
Start Nginx Service
        ↓
Enable Nginx at Boot
        ↓
Verify Service Status
        ↓
Display Result
        ↓
Execute Handler (if required)
```

---

# Play Definition

```yaml
- name: Configure Nginx Web Server
```

Purpose:

Provides a descriptive name for the play.

Displayed during execution:

```text
PLAY [Configure Nginx Web Server]
```

---

# Target Hosts

```yaml
hosts: webservers
```

Purpose:

Specifies the inventory group on which the playbook will run.

Example inventory:

```ini
[webservers]
192.168.1.100
192.168.1.101
```

The playbook will run on both servers.

---

# Privilege Escalation

```yaml
become: yes
```

Purpose:

Runs all tasks using sudo privileges.

Equivalent Linux command:

```bash
sudo <command>
```

Required because package installation, configuration changes, and service management need root permissions.

---

# Variables Section

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

* Easier maintenance
* Avoid hardcoding values
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
- name: Create deployment user
  user:
    name: "{{ deploy_user }}"
    shell: /bin/bash
    create_home: yes
    state: present
```

Purpose:

Creates a dedicated user account for deployments.

Equivalent Linux command:

```bash
sudo useradd -m -s /bin/bash deploy
```

Benefits:

* Better security
* Dedicated deployment account
* Easier permission management

Expected Result:

```text
User deploy exists on the server
```

---

# Task 2: Update Package Cache

```yaml
- name: Update package cache
  apt:
    update_cache: yes
```

Purpose:

Updates package repository information.

Equivalent Command:

```bash
sudo apt update
```

Why Needed:

Ensures the latest package information is available before installation.

---

# Task 3: Install Nginx

```yaml
- name: Install nginx
  apt:
    name: "{{ web_package }}"
    state: present
```

Purpose:

Installs Nginx if not already installed.

Equivalent Command:

```bash
sudo apt install nginx -y
```

Idempotent Behavior:

* Installs if missing
* Skips if already installed

---

# Task 4: Create Website Directory

```yaml
- name: Create website directory
  file:
    path: "{{ website_root }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
```

Purpose:

Ensures the website root directory exists.

Equivalent Command:

```bash
sudo mkdir -p /var/www/html
sudo chown www-data:www-data /var/www/html
sudo chmod 755 /var/www/html
```

---

# Task 5: Deploy Website Page

```yaml
- name: Deploy website page
  copy:
    src: files/index.html
    dest: "{{ website_root }}/index.html"
```

Purpose:

Copies the website content from the Ansible control node to the target server.

Equivalent Command:

```bash
scp index.html server:/var/www/html/index.html
```

Result:

The custom webpage becomes available through Nginx.

---

# Notification Trigger

```yaml
notify:
  - restart nginx
```

Purpose:

If the webpage changes, Ansible notifies the handler.

This prevents unnecessary service restarts.

---

# Task 6: Deploy Nginx Configuration

```yaml
- name: Deploy nginx configuration
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/sites-available/default
```

Purpose:

Creates an Nginx configuration using Jinja2 variables.

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
- name: Start nginx service
  service:
    name: "{{ web_service }}"
    state: started
```

Purpose:

Ensures Nginx is running.

Equivalent Command:

```bash
sudo systemctl start nginx
```

---

# Task 8: Enable Nginx Service

```yaml
- name: Enable nginx service
  service:
    name: "{{ web_service }}"
    enabled: yes
```

Purpose:

Ensures Nginx automatically starts after a reboot.

Equivalent Command:

```bash
sudo systemctl enable nginx
```

---

# Task 9: Verify Nginx Status

```yaml
- name: Verify nginx is active
  command: systemctl is-active nginx
  register: nginx_status
  changed_when: false
```

Purpose:

Checks if Nginx is active.

Equivalent Command:

```bash
systemctl is-active nginx
```

Sample Output:

```text
active
```

The result is stored in:

```yaml
nginx_status
```

---

# Task 10: Display Verification Result

```yaml
- name: Display nginx status
  debug:
    msg: "Nginx Status: {{ nginx_status.stdout }}"
```

Purpose:

Prints the status to the console.

Example Output:

```text
TASK [Display nginx status]

ok: [server] =>
{
  "msg": "Nginx Status: active"
}
```

---

# Handler Section

```yaml
handlers:

  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

Purpose:

Restarts Nginx only when configuration files or website content changes.

Equivalent Command:

```bash
sudo systemctl restart nginx
```

Benefits:

* Avoids unnecessary restarts
* Reduces downtime
* Follows production best practices

---

# Expected End State

After successful execution:

✓ Deployment user created

✓ Package cache updated

✓ Nginx installed

✓ Website directory created

✓ Custom webpage deployed

✓ Nginx configuration applied

✓ Nginx running

✓ Nginx enabled at boot

✓ Service verification completed

✓ Automatic restart on configuration changes

---

# Technologies Used

* Ansible
* Linux
* Nginx
* YAML
* Jinja2 Templates
* SSH
* Systemd

# Learning Outcomes

This project demonstrates practical knowledge of:

* Ansible Playbooks
* Variables
* User Management
* Package Management
* File Management
* Templates
* Handlers
* Service Management
* Verification Tasks
* Idempotency
* Infrastructure Automation
