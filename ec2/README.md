# AWS EC2 Multi-OS Provisioning using Ansible

## Project Overview

This project demonstrates how to provision multiple AWS EC2 instances with different operating systems using Ansible.

The project showcases the following Ansible concepts:

* Variables
* Loops
* AWS EC2 Provisioning
* Ansible Vault
* Dynamic Resource Creation
* Task Reusability
* Infrastructure as Code (IaC)

The playbook creates multiple EC2 instances in AWS from a single task using variables and loops.

---

# Project Objectives

Provision the following EC2 instances:

| Instance Name      | Operating System           | Instance Type |
| ------------------ | -------------------------- | ------------- |
| Ubuntu-Server      | Ubuntu 24.04               | t2.micro      |
| AmazonLinux-Server | Amazon Linux 2023          | t2.micro      |
| RHEL-Server        | Red Hat Enterprise Linux 9 | t2.micro      |

The project also includes a separate task file to stop the provisioned instances.

---

# Technologies Used

* Ansible
* AWS EC2
* Ansible Vault
* AWS IAM
* Python boto3
* Python botocore

---

# Project Structure

```text
aws-ec2-provisioning/

├── create-ec2.yml
├── stop-ec2.yml

├── group_vars/
│   └── all/
│       ├── vars.yml
│       └── pass.yml

├── tasks/
│   └── stop_instances.yml

└── README.md
```

---

# Prerequisites

## Install Ansible

```bash
sudo apt update
sudo apt install ansible -y
```

Verify:

```bash
ansible --version
```

---

## Install AWS Collection

```bash
ansible-galaxy collection install amazon.aws
```

Verify:

```bash
ansible-galaxy collection list
```

---

## Install Python Dependencies

```bash
pip install boto3 botocore
```

Verify:

```bash
pip list | grep boto
```

---

# Variables Configuration

## group_vars/all/vars.yml

This file contains all configurable parameters.

```yaml
---
aws_region: ap-south-1

key_name: my-keypair

ec2_instances:

  - name: Ubuntu-Server
    image_id: ami-xxxxxxxxxxxx
    instance_type: t2.micro

  - name: AmazonLinux-Server
    image_id: ami-yyyyyyyyyyyy
    instance_type: t2.micro

  - name: RHEL-Server
    image_id: ami-zzzzzzzzzzzz
    instance_type: t2.micro
```

### Variable Description

| Variable      | Purpose           |
| ------------- | ----------------- |
| aws_region    | AWS Region        |
| key_name      | EC2 SSH Key Pair  |
| name          | EC2 Name Tag      |
| image_id      | AMI ID            |
| instance_type | EC2 Instance Type |

---

# Secure Credential Management

## group_vars/all/pass.yml

Stores AWS credentials.

```yaml
---
aws_access_key: YOUR_ACCESS_KEY
aws_secret_key: YOUR_SECRET_KEY
```

Encrypt the file:

```bash
ansible-vault encrypt group_vars/all/pass.yml
```

View encrypted file:

```bash
cat group_vars/all/pass.yml
```

Output:

```text
$ANSIBLE_VAULT;1.1;AES256
xxxxxxxxxxxxxxxxxxxx
```

---

# EC2 Creation Playbook

## create-ec2.yml

Purpose:

Creates multiple EC2 instances using a loop.

Workflow:

```text
Read Variables
      ↓
Authenticate with AWS
      ↓
Loop Through Instance List
      ↓
Create Ubuntu Instance
      ↓
Create Amazon Linux Instance
      ↓
Create RHEL Instance
      ↓
Display Instance Details
```

### Key Features

* Uses Variables
* Uses Loops
* Creates Multiple EC2 Instances
* Reusable Code
* No Hardcoded Values

---

# Loop Implementation

Example:

```yaml
loop: "{{ ec2_instances }}"
```

Execution:

```text
Iteration 1 → Ubuntu-Server

Iteration 2 → AmazonLinux-Server

Iteration 3 → RHEL-Server
```

Benefits:

* Less code
* Easier maintenance
* Better scalability

---

# Stop EC2 Instances

## stop-ec2.yml

Purpose:

Stops only the EC2 instances defined in `vars.yml`.

Workflow:

```text
Read Instance Names
        ↓
Search Instances By Name Tag
        ↓
Fetch Instance IDs
        ↓
Stop Instances
        ↓
Display Status
```

---

# Reusable Task File

## tasks/stop_instances.yml

Purpose:

Contains reusable tasks to stop EC2 instances.

Benefits:

* Cleaner playbook structure
* Reusable automation logic
* Easier maintenance

---

# Running the Project

## Create EC2 Instances

```bash
ansible-playbook create-ec2.yml --ask-vault-pass
```

---

## Stop EC2 Instances

```bash
ansible-playbook stop-ec2.yml --ask-vault-pass
```

---

# Sample Output

## EC2 Creation

```text
TASK [Launch EC2 instances]

changed: [localhost]

Instance Name: Ubuntu-Server
Public IP: 13.232.10.100

Instance Name: AmazonLinux-Server
Public IP: 15.206.20.200

Instance Name: RHEL-Server
Public IP: 43.205.30.150
```

---

## EC2 Stop

```text
TASK [Stop EC2 instances]

changed: [localhost]

Ubuntu-Server stopped

AmazonLinux-Server stopped

RHEL-Server stopped
```

---

# Ansible Concepts Demonstrated

| Concept                | Implementation          |
| ---------------------- | ----------------------- |
| Variables              | vars.yml                |
| Vault                  | pass.yml                |
| Loops                  | loop over ec2_instances |
| AWS Modules            | amazon.aws.ec2_instance |
| Reusable Tasks         | stop_instances.yml      |
| Tags                   | EC2 Name Tags           |
| Infrastructure as Code | AWS Provisioning        |
| Debug                  | Output instance details |

---

# Learning Outcomes

After completing this project, you will understand:

* AWS EC2 provisioning using Ansible
* Managing secrets using Ansible Vault
* Using variables for reusable automation
* Looping through complex data structures
* Organizing Ansible projects
* Creating reusable task files
* Managing EC2 lifecycle operations
* Infrastructure as Code best practices

---

# Future Enhancements

* Add Security Groups
* Add Subnets
* Create VPC using Ansible
* Dynamic Inventory
* Start/Stop/Reboot Playbooks
* EC2 Termination Playbook
* Multi-Region Deployment
* Jinja2 Templates
* Ansible Roles
* CI/CD Integration with Jenkins or GitHub Actions

---

# Author

Soumya Hanasi

Project: AWS EC2 Multi-OS Provisioning using Ansible

Skills Demonstrated:

* Ansible
* AWS EC2
* Infrastructure Automation
* Ansible Vault
* Variables and Loops
* DevOps Practices
