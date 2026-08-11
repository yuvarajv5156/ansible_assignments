# Assignment 4 – Install and Configure Nginx Using Ansible

## Objective

Install Nginx on Ubuntu EC2 instances using Ansible, change the default Nginx port from `80` to `8081`, deploy an HTML application, and verify the application.

## Project Structure

```text
assignment4/
├── ansible.cfg
├── inventory
├── playbook.yml
├── index.html
└── README.md
```

## Prerequisites

* Ubuntu EC2 instance(s)
* Ansible installed on the control node
* SSH access to the target EC2 instance(s)
* EC2 Security Group allowing SSH port `22`
* EC2 Security Group allowing application port `8081`

## Step 1: Create Assignment Directory

```bash
cd ~/ansible_assignments
mkdir assignment4
cd assignment4
```

## Step 2: Create Ansible Configuration

Create `ansible.cfg`:

```bash
nano ansible.cfg
```

Configuration:

```ini
[defaults]
inventory = inventory
host_key_checking = False
```

## Step 3: Create Inventory

Create the inventory file:

```bash
nano inventory
```

Example:

```ini
[webservers]
webserver1 ansible_host=3.88.41.72
webserver2 ansible_host=34.228.41.85

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/yuvaraj-aws.pem
```

Update the IP addresses and SSH key path according to the EC2 environment.

## Step 4: Create HTML Application

Create `index.html`:

```bash
nano index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Nginx Demo</title>
</head>
<body>
    <h1>Nginx deployed using Ansible</h1>
    <p>Application is running on port 8081</p>
</body>
</html>
```

## Step 5: Create Ansible Playbook

Create `playbook.yml`:

```bash
nano playbook.yml
```

Add:

```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Change Nginx port from 80 to 8081
      ansible.builtin.replace:
        path: /etc/nginx/sites-available/default
        regexp: 'listen 80 default_server;'
        replace: 'listen 8081 default_server;'

    - name: Deploy application
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Restart Nginx
      ansible.builtin.systemd:
        name: nginx
        state: restarted
        enabled: true
```

## Step 6: Test Ansible Connectivity

Run:

```bash
ansible all -m ping
```

Expected result:

```text
webserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

webserver2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Step 7: Run the Playbook

Run:

```bash
ansible-playbook playbook.yml
```

The playbook performs the following tasks:

1. Installs Nginx.
2. Changes the Nginx port from `80` to `8081`.
3. Copies `index.html` to `/var/www/html/`.
4. Sets the correct ownership and permissions.
5. Restarts and enables Nginx.

## Step 8: Verify Nginx Port

Run:

```bash
ansible webservers -a "sudo ss -tlnp | grep 8081"
```

Nginx should be listening on port `8081`.

## Step 9: Verify Application

From the control node:

```bash
curl http://3.88.41.72:8081
```

or:

```bash
curl http://34.228.41.85:8081
```

Expected output contains:

```text
Nginx deployed using Ansible
Application is running on port 8081
```

The application can also be accessed from a browser:

```text
http://<EC2-PUBLIC-IP>:8081
```

## Step 10: Verify Nginx Service

Run:

```bash
ansible webservers -a "systemctl status nginx"
```

Nginx should be in the `active (running)` state.

## Step 11: Git Commands

Check the files:

```bash
git status
```

Add Assignment 4:

```bash
git add assignment4/
```

Commit:

```bash
git commit -m "Add Assignment 4 Nginx Ansible deployment"
```

Push to GitHub:

```bash
git push origin main
```

## Result

Nginx was successfully installed and configured on the Ubuntu EC2 instances using Ansible. The application was deployed and verified on port `8081`.
