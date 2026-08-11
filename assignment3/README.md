# Assignment 3 – Ansible Dynamic Inventory with AWS EC2

## Objective

Configure **Ansible Dynamic Inventory** to automatically discover AWS EC2 instances instead of manually maintaining an inventory file.

### Example

If AWS has:

```text
3 EC2 instances
        ↓
Dynamic Inventory discovers 3
```

If Auto Scaling launches 2 more:

```text
5 EC2 instances
        ↓
Dynamic Inventory discovers 5
```

If 1 instance is terminated:

```text
4 EC2 instances
        ↓
Dynamic Inventory discovers 4
```

No manual inventory update is required.

---

# Architecture

```text
AWS EC2 Instances
       |
       | EC2 Tags
       | env=webserver
       | env=appserver
       | env=dbserver
       |
       v
amazon.aws.aws_ec2
Dynamic Inventory Plugin
       |
       v
aws_ec2.yml
       |
       v
Ansible
       |
       v
Playbook
       |
       v
Install / Configure Applications
```

---

# Prerequisites

* AWS account
* EC2 instances
* Ubuntu EC2 instance for Ansible control node
* Python 3
* Ansible
* AWS CLI
* IAM Role attached to Ansible EC2 instance
* SSH connectivity from Ansible control node to target EC2 instances

---

# 1. Install Python and Virtual Environment

Update the package repository:

```bash
sudo apt update
```

Install Python and virtual environment packages:

```bash
sudo apt install python3-full python3-venv -y
```

---

# 2. Create Python Virtual Environment

Create the virtual environment:

```bash
python3 -m venv ~/ansible-venv
```

Activate it:

```bash
source ~/ansible-venv/bin/activate
```

The terminal should show:

```text
(ansible-venv) ubuntu@ip-xxx:~$
```

---

# 3. Install boto3 and botocore

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

Install AWS Python libraries:

```bash
python -m pip install boto3 botocore
```

Verify boto3:

```bash
python -c "import boto3; print(boto3.__version__)"
```

---

# 4. Install AWS Ansible Collection

Install the Amazon AWS collection:

```bash
ansible-galaxy collection install amazon.aws
```

Verify:

```bash
ansible-galaxy collection list | grep amazon.aws
```

---

# 5. Create IAM Role

Create an IAM role and attach it to the **EC2 instance where Ansible is running**.

The role requires permission to discover EC2 instances.

Example IAM policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeTags",
        "ec2:DescribeRegions"
      ],
      "Resource": "*"
    }
  ]
}
```

Attach this IAM role to the Ansible control EC2 instance.

---

# 6. Verify IAM Role

Check whether the EC2 instance can access AWS:

```bash
aws sts get-caller-identity
```

If the IAM role is configured correctly, AWS account and role information will be displayed.

---

# 7. Verify EC2 Access

Run:

```bash
aws ec2 describe-instances
```

If EC2 information is displayed, the AWS CLI can access EC2 using the IAM role.

---

# 8. Add Tags to EC2 Instances

Use a single tag key called `env`.

Example:

| EC2 Instance | Key | Value     |
| ------------ | --- | --------- |
| EC2-1        | env | webserver |
| EC2-2        | env | appserver |
| EC2-3        | env | dbserver  |

In the AWS Console:

```text
EC2
 → Instances
 → Select Instance
 → Tags
 → Manage tags
 → Add new tag
```

Example:

```text
Key   = env
Value = webserver
```

For another instance:

```text
Key   = env
Value = appserver
```

For another instance:

```text
Key   = env
Value = dbserver
```

These tag values will automatically become Ansible groups.

For example:

```text
env=webserver
        ↓
Ansible group: webserver
```

---

# 9. Create Dynamic Inventory File

Create the file:

```bash
vi aws_ec2.yml
```

Add:

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1

keyed_groups:
  - key: tags.env
    prefix: ""
    separator: ""
```

> Change `us-east-1` if your EC2 instances are in another AWS region.

For example, Mumbai:

```yaml
regions:
  - ap-south-1
```

---

# 10. Test Dynamic Inventory

Run:

```bash
ansible-inventory -i aws_ec2.yml --graph
```

Example output:

```text
@all:
  |--@webserver:
  |  |--ec2-13-217-88-167.compute-1.amazonaws.com
  |--@appserver:
  |  |--ec2-xx-xx-xx-xx.compute-1.amazonaws.com
  |--@dbserver:
     |--ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

This confirms that Ansible is automatically discovering EC2 instances from AWS tags.

---

# 11. View Complete Dynamic Inventory

Run:

```bash
ansible-inventory -i aws_ec2.yml --list
```

This displays the complete inventory in JSON format.

---

# 12. Create ansible.cfg

Create:

```bash
vi ansible.cfg
```

Add:

```ini
[defaults]
inventory = /home/ubuntu/ansible_assignments/assignment3/inventory/aws_ec2.yml
host_key_checking = False
private_key_file = /home/ubuntu/ansible_assignments/assignment3/yuvaraj-aws.pem
remote_user = ubuntu
```

Now Ansible automatically uses `aws_ec2.yml` as the inventory.

Therefore, instead of:

```bash
ansible -i aws_ec2.yml all -m ping
```

we can use:

```bash
ansible all -m ping
```

---

# 13. Test Connectivity

Run:

```bash
ansible all -m ping
```

Expected output:

```text
SUCCESS
"ping": "pong"
```

This confirms that Ansible can connect to the dynamically discovered EC2 instances.

---

# 14. Create Playbook

For this demo, use the Nginx installation playbook from Assignment 1.

Create:

```bash
vi test.yml
```

Example:

```yaml
---
- name: Install Nginx
  hosts: webserver
  become: true

  tasks:

    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

The important part is:

```yaml
hosts: webserver
```

The `webserver` group is created automatically from:

```text
env=webserver
```

in AWS.

---

# 15. Run the Playbook

Run:

```bash
ansible-playbook test.yml
```

Expected result:

```text
PLAY RECAP

ec2-xx-xx-xx-xx.compute-1.amazonaws.com :
ok=3
changed=0
unreachable=0
failed=0
```

If:

```text
failed=0
```

the playbook completed successfully.

---

# 16. Verify Nginx

Connect to the target EC2 instance and check:

```bash
systemctl status nginx
```

Or test from the Ansible server:

```bash
ansible webserver -m shell -a "systemctl is-active nginx" --become
```

Expected:

```text
active
```

---

# 17. Dynamic Inventory Demonstration

### Initial State

Suppose AWS has:

```text
EC2-1 → env=webserver
EC2-2 → env=appserver
EC2-3 → env=dbserver
```

Run:

```bash
ansible-inventory --graph
```

Ansible discovers all three instances.

---

### Launch a New EC2 Instance

Launch another EC2 instance and add:

```text
Key   = env
Value = webserver
```

Do not modify:

```text
aws_ec2.yml
```

Run:

```bash
ansible-inventory --graph
```

The new EC2 instance is automatically discovered under:

```text
webserver
```

---

### Terminate an EC2 Instance

Terminate one of the existing EC2 instances.

Run:

```bash
ansible-inventory --graph
```

The terminated instance will no longer appear in the dynamic inventory.

---

# 18. Important Difference

## Static Inventory

With static inventory, EC2 instances are manually added:

```ini
[webserver]
10.0.1.10
10.0.1.11
10.0.1.12
```

If a new EC2 instance is launched, the inventory must be manually updated.

---

## Dynamic Inventory

With dynamic inventory:

```text
AWS EC2
   ↓
amazon.aws.aws_ec2 plugin
   ↓
Automatically discovers instances
   ↓
Creates groups from EC2 tags
   ↓
Ansible
```

No manual IP address maintenance is required.

---

# 19. Important Commands

### Check AWS identity

```bash
aws sts get-caller-identity
```

### Check EC2 instances

```bash
aws ec2 describe-instances
```

### Check installed AWS collection

```bash
ansible-galaxy collection list | grep amazon.aws
```

### View dynamic inventory

```bash
ansible-inventory -i aws_ec2.yml --graph
```

### View complete inventory

```bash
ansible-inventory -i aws_ec2.yml --list
```

### Test connectivity

```bash
ansible all -m ping
```

### Run playbook

```bash
ansible-playbook test.yml
```

---

# Conclusion

Ansible Dynamic Inventory automatically discovers EC2 instances from AWS.

In this assignment, EC2 instances are grouped using the AWS tag:

```text
env
```

For example:

```text
env=webserver
```

automatically creates the Ansible group:

```text
webserver
```

The major advantage is that when EC2 instances are launched or terminated, the Ansible inventory automatically reflects the current AWS environment without manually editing an inventory file.
