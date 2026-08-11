# Assignment 5 – Install Tomcat 10 and Change Port Using Ansible

## Objective

Install Tomcat 10 on an Ubuntu EC2 server using Ansible, change the default Tomcat port from `8080` to `9090`, restart the Tomcat service, and verify the configuration.

## Project Structure

```text
assignment5/
├── ansible.cfg
├── inventory
├── playbook.yml
└── README.md
```

## Prerequisites

* Ubuntu EC2 instance
* Ansible installed on the control node
* SSH access to the Tomcat EC2 server
* SSH private key (`.pem`)
* EC2 Security Group allowing SSH port `22`
* EC2 Security Group allowing Tomcat port `9090`

## Step 1: Create Assignment Directory

```bash
cd ~/ansible_assignments
mkdir assignment5
cd assignment5
```

## Step 2: Create Ansible Configuration

Create `ansible.cfg`:

```bash
nano ansible.cfg
```

Add:

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
[tomcatserver]
tomcat1 ansible_host=<TOMCAT-EC2-PUBLIC-IP>

[tomcatserver:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/yuvaraj-aws.pem
```

Replace `<TOMCAT-EC2-PUBLIC-IP>` with the actual EC2 public IP address.

Update the SSH private key path if required.

## Step 4: Create the Ansible Playbook

Create `playbook.yml`:

```bash
nano playbook.yml
```

Add:

```yaml
---
- name: Install Tomcat 10 and change port
  hosts: tomcatserver
  become: true

  tasks:
    - name: Install Tomcat 10
      ansible.builtin.apt:
        name: tomcat10
        state: present
        update_cache: yes

    - name: Change Tomcat port from 8080 to 9090
      ansible.builtin.replace:
        path: /etc/tomcat10/server.xml
        regexp: 'port="8080"'
        replace: 'port="9090"'

    - name: Restart Tomcat service
      ansible.builtin.systemd:
        name: tomcat10
        state: restarted
        enabled: true
```

## Step 5: Test Ansible Connectivity

Run:

```bash
ansible all -m ping
```

Expected result:

```text
tomcat1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Step 6: Run the Playbook

Run:

```bash
ansible-playbook playbook.yml
```

The playbook performs the following tasks:

1. Installs Tomcat 10.
2. Changes the Tomcat connector port from `8080` to `9090`.
3. Restarts Tomcat.
4. Enables Tomcat to start automatically at boot.

## Step 7: Verify Tomcat Service

Run:

```bash
ansible tomcatserver -a "systemctl status tomcat10"
```

Tomcat should show:

```text
Active: active (running)
```

## Step 8: Verify Tomcat Port

Run:

```bash
ansible tomcatserver -a "sudo ss -tlnp | grep 9090"
```

Expected output should show Tomcat listening on port `9090`.

You can also verify the configuration:

```bash
ansible tomcatserver -a "grep -n 'port=\"9090\"' /etc/tomcat10/server.xml"
```

## Step 9: Test Tomcat from Browser

Allow TCP port `9090` in the EC2 Security Group.

Then open:

```text
http://<TOMCAT-EC2-PUBLIC-IP>:9090
```

The Tomcat default page should be displayed.

## Step 10: Verify Tomcat Automatically Starts

Run:

```bash
ansible tomcatserver -a "systemctl is-enabled tomcat10"
```

Expected:

```text
enabled
```

## Step 11: Git Commands

Check the assignment:

```bash
git status
```

Add Assignment 5:

```bash
git add assignment5/
```

Commit:

```bash
git commit -m "Add Assignment 5 Tomcat Ansible deployment"
```

Push to GitHub:

```bash
git push origin main
```

## Result

Tomcat 10 was successfully installed and configured using Ansible. The default Tomcat port was changed from `8080` to `9090`, the service was restarted and enabled, and the Tomcat application was verified on port `9090`.
