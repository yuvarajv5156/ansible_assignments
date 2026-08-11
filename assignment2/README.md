# Assignment 2 – Ansible Vault

## Objective

The objective of this assignment is to demonstrate how to use **Ansible Vault** to securely store sensitive information such as database usernames and passwords.

## Project Structure

```text
assignment2/
├── ansible.cfg
├── inventory
├── deploy.yml
└── secrets.yml
```

## 1. Ansible Configuration

The `ansible.cfg` file contains the Ansible configuration.

```ini
[defaults]
inventory = /home/ubuntu/assignment2/inventory
remote_user = ubuntu
private_key_file = /home/ubuntu/assignment2/yuvaraj-aws.pem
host_key_checking = False
retry_files_enabled = False
```

## 2. Inventory

The `inventory` file contains the target EC2 instance.

```text
54.227.201.134
```

## 3. SSH Private Key

The EC2 private key must have secure permissions.

```bash
chmod 400 /home/ubuntu/assignment2/yuvaraj-aws.pem
```

Verify:

```bash
ls -l /home/ubuntu/assignment2/yuvaraj-aws.pem
```

The key should not be accessible by other users.

## 4. Create Ansible Vault

Create an encrypted variables file:

```bash
ansible-vault create secrets.yml
```

Example contents:

```yaml
db_username: admin
db_password: MySecret@123
```

The file is encrypted after saving.

Check the encrypted file:

```bash
cat secrets.yml
```

## 5. View Vault File

To view the decrypted contents:

```bash
ansible-vault view secrets.yml
```

## 6. Playbook

The `deploy.yml` playbook loads the encrypted variables using `vars_files`.

```yaml
---
- name: Deploy application
  hosts: all
  become: true

  vars_files:
    - secrets.yml

  tasks:
    - name: Show database username
      ansible.builtin.debug:
        msg: "Database user is {{ db_username }}"

    - name: Use database password
      ansible.builtin.debug:
        msg: "Database password is {{ db_password }}"
```

## 7. Test Ansible Connectivity

Before running the playbook, verify SSH connectivity:

```bash
ansible all -m ping
```

Expected output:

```text
54.227.201.134 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 8. Run the Playbook

Run the playbook and provide the Vault password:

```bash
ansible-playbook deploy.yml --ask-vault-pass
```

Ansible will prompt:

```text
Vault password:
```

Enter the password used when creating `secrets.yml`.

## 9. Edit Vault File

To modify the encrypted file:

```bash
ansible-vault edit secrets.yml
```

## 10. Change Vault Password

To change the Vault password:

```bash
ansible-vault rekey secrets.yml
```

## Important Ansible Vault Commands

| Command                            | Purpose               |
| ---------------------------------- | --------------------- |
| `ansible-vault create secrets.yml` | Create encrypted file |
| `ansible-vault view secrets.yml`   | View encrypted file   |
| `ansible-vault edit secrets.yml`   | Edit encrypted file   |
| `ansible-vault encrypt file.yml`   | Encrypt existing file |
| `ansible-vault decrypt file.yml`   | Decrypt file          |
| `ansible-vault rekey secrets.yml`  | Change Vault password |

## Git Commands

Check the files:

```bash
git status
```

Add the README:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Update Ansible Vault README"
```

Push:

```bash
git push origin main
```

## Security Note

Do **not** commit the unencrypted passwords or private SSH keys to Git.

The `.pem` file should be excluded from Git using `.gitignore`.

Example:

```text
*.pem
```

Ansible Vault keeps sensitive variables encrypted while allowing Ansible to use them securely during playbook execution.
