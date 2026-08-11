# Ansible Assignment 1 – Tomcat Installation and Application Deployment

## Overview

This project demonstrates how to use **Ansible Roles** to:

1. Install Apache Tomcat 10.1.57 on an Ubuntu EC2 instance.
2. Deploy a WAR application to Tomcat.
3. Backup the existing WAR before deployment.
4. Start Tomcat and verify application health.

## Project Structure

```text
assignment1/
├── ansible.cfg
├── inventory
├── site.yml
└── roles/
    ├── tomcat/
    │   ├── defaults/
    │   │   └── main.yml
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── handlers/
    │   │   └── main.yml
    │   └── templates/
    │       └── tomcat.service.j2
    │
    └── tomcat_deploy_v2/
        ├── defaults/
        │   └── main.yml
        ├── files/
        │   └── book-seller.war
        └── tasks/
            └── main.yml
```

# Role 1 – Tomcat Installation

Role name:

```text
roles/tomcat
```

This role installs and configures Apache Tomcat.

### Tasks performed

* Install Java 17
* Install required packages
* Create `tomcat` group
* Create `tomcat` user
* Create Tomcat installation directory
* Download Tomcat 10.1.57 binary
* Extract Tomcat
* Create `current` symbolic link
* Set Tomcat ownership
* Create systemd service
* Start and enable Tomcat

### Tomcat Installation Directory

```text
/opt/tomcat
```

Tomcat is accessed through:

```text
/opt/tomcat/current
```

### Verify Tomcat

```bash
sudo systemctl status tomcat
```

Check port 8080:

```bash
sudo ss -lntp | grep 8080
```

Test Tomcat:

```bash
curl -I http://localhost:8080/
```

# Role 2 – Tomcat Application Deployment

Role name:

```text
roles/tomcat_deploy_v2
```

This role is used to deploy the application WAR file.

WAR file:

```text
roles/tomcat_deploy_v2/files/book-seller.war
```

## Deployment Steps

### 1. Create Backup Directory

```text
/opt/tomcat/backup
```

### 2. Stop Tomcat

The role stops the Tomcat service before deployment.

```text
tomcat → stopped
```

### 3. Check Existing WAR

The `stat` module checks whether the existing WAR file is available.

```text
/opt/tomcat/current/webapps/book-seller.war
```

### 4. Backup Existing WAR

If the existing WAR exists, it is backed up to:

```text
/opt/tomcat/backup/book-seller.war.backup
```

### 5. Remove Old WAR

The old WAR is removed from:

```text
/opt/tomcat/current/webapps/
```

### 6. Remove Extracted Application

The previously extracted application directory is removed:

```text
/opt/tomcat/current/webapps/book-seller/
```

### 7. Deploy New WAR

The new WAR is copied to:

```text
/opt/tomcat/current/webapps/book-seller.war
```

### 8. Start Tomcat

The Tomcat service is started and enabled.

### 9. Wait for Tomcat

Ansible waits for port `8080` to become available.

### 10. Application Health Check

The application is checked using:

```text
http://localhost:8080/book-seller/
```

The Ansible `uri` module verifies that the application returns:

```text
HTTP 200
```

# Playbook

The `site.yml` calls both roles.

Example:

```yaml
---
- name: Deploying version2
  hosts: tomcat
  become: yes

  roles:
    - tomcat
    - tomcat_deploy_v2
```

# Ansible Configuration

The project uses `ansible.cfg` to define the inventory, remote user, and SSH private key.

Example:

```ini
[defaults]
inventory = inventory
remote_user = ubuntu
private_key_file = /home/ubuntu/yuvaraj-aws.pem
host_key_checking = False
```

**Do not commit the private `.pem` file to Git.**

# Inventory

Example:

```ini
[tomcat]
54.227.201.134
```

Replace the IP address with the target EC2 instance IP.

# Run the Playbook

## Syntax Check

```bash
ansible-playbook site.yml --syntax-check
```

## Deploy

```bash
ansible-playbook site.yml
```

# Deployment Result

The deployment was successfully completed.

```text
ok=12
changed=5
unreachable=0
failed=0
skipped=0
```

Application health check:

```text
Application deployed successfully. HTTP status: 200
```

Application URL:

```text
http://<EC2-PUBLIC-IP>:8080/book-seller/
```

# Git Commands

Check changes:

```bash
git status
```

Add README:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Update README with Tomcat deployment steps"
```

Push:

```bash
git push origin main
```

# Security

Do not commit SSH private keys.

Add the following to `.gitignore`:

```text
*.pem
*.key
*.retry
```

## Summary

This project demonstrates an Ansible-based Tomcat deployment process using two reusable roles:

```text
tomcat
   ↓
Install and configure Tomcat
   ↓
tomcat_deploy_v2
   ↓
Stop Tomcat
   ↓
Backup existing WAR
   ↓
Deploy new WAR
   ↓
Start Tomcat
   ↓
Health Check
   ↓
HTTP 200 – Deployment Successful
```
