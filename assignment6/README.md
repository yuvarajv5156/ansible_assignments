# Assignment 6 – Target Instances Report Using Ansible

## Objective

Write an Ansible playbook to collect and display system information from target instances.

The report includes:

* Hostname
* CPU
* RAM
* Storage

## Project Structure

```text
assignment6/
├── ansible.cfg
├── inventory
├── instance_report.yml
└── README.md
```

## Prerequisites

* Ansible installed on the control node
* Ubuntu target EC2 instances
* SSH connectivity between the Ansible control node and target instances
* SSH private key (`.pem`)
* `become` privileges on target instances

## Step 1: Create Assignment Directory

```bash
cd ~/ansible_assignments
mkdir assignment6
cd assignment6
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

Create the inventory:

```bash
nano inventory
```

Example:

```ini
[all]
server1 ansible_host=<TARGET-IP-1>
server2 ansible_host=<TARGET-IP-2>

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/yuvaraj-aws.pem
```

Replace the target IP addresses and SSH key path with the actual values.

## Step 4: Create the Instance Report Playbook

Create the playbook:

```bash
nano instance_report.yml
```

Add:

```yaml
---
- name: Get Target Instances Report
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Display Hostname
      ansible.builtin.debug:
        msg: "Hostname: {{ ansible_hostname }}"

    - name: Display CPU
      ansible.builtin.debug:
        msg: "CPU: {{ ansible_processor_vcpus }} cores"

    - name: Display RAM
      ansible.builtin.debug:
        msg: "RAM: {{ ansible_memtotal_mb }} MB"

    - name: Display Storage
      ansible.builtin.debug:
        msg: "Storage: {{ ansible_mounts[0].size_total | human_readable }}"
```

## Step 5: Test Ansible Connectivity

Run:

```bash
ansible all -m ping
```

Expected result:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

server2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Step 6: Run the Instance Report

Run:

```bash
ansible-playbook instance_report.yml
```

The playbook gathers facts from all target instances and displays:

```text
Hostname
CPU
RAM
Storage
```

Example output:

```text
TASK [Display Hostname]
ok: [server1] => {
    "msg": "Hostname: ip-172-31-59-104"
}

TASK [Display CPU]
ok: [server1] => {
    "msg": "CPU: 2 cores"
}

TASK [Display RAM]
ok: [server1] => {
    "msg": "RAM: 1950 MB"
}

TASK [Display Storage]
ok: [server1] => {
    "msg": "Storage: 8.00 GB"
}
```

## Step 7: Verify Gathered Facts

You can verify the CPU information with:

```bash
ansible all -m setup -a "filter=ansible_processor_vcpus"
```

Verify RAM:

```bash
ansible all -m setup -a "filter=ansible_memtotal_mb"
```

Verify hostname:

```bash
ansible all -m setup -a "filter=ansible_hostname"
```

## Step 8: Git Commands

Check the assignment files:

```bash
git status
```

Add Assignment 6:

```bash
git add assignment6/
```

Commit the changes:

```bash
git commit -m "Add Assignment 6 target instances report"
```

Push to GitHub:

```bash
git push origin main
```

## Result

The Ansible playbook successfully gathers system facts from the target instances and displays the **hostname, CPU, RAM, and storage** information.
