# Ansible Tutorial: From Basics to Web Server Deployment

## Table of Contents
1. [What is Ansible?](#what-is-ansible)
2. [Ansible Architecture](#ansible-architecture)
3. [Important Files and Configurations](#important-files-and-configurations)
4. [Connectivity Setup](#connectivity-setup)
5. [Use Case: Installing a Web Server](#use-case-installing-a-web-server)

---

## What is Ansible?

Ansible is an open-source automation tool used for:
- **Configuration Management**: Managing system configurations
- **Application Deployment**: Deploying applications across multiple servers
- **Orchestration**: Coordinating multiple automated tasks
- **Provisioning**: Setting up infrastructure

**Key Features:**
- Agentless (uses SSH for Linux, WinRM for Windows)
- Uses YAML for playbooks (human-readable)
- Idempotent (running the same playbook multiple times produces the same result)
- Push-based architecture

---

## Ansible Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CONTROL NODE                             │
│  ┌────────────────────────────────────────────────────┐     │
│  │  Ansible Engine                                     │     │
│  │  • Playbooks (YAML)                                 │     │
│  │  • Inventory (hosts file)                           │     │
│  │  • Modules (pre-built automation units)             │     │
│  │  • Plugins                                          │     │
│  └────────────────────────────────────────────────────┘     │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ SSH (Linux/Unix)
                        │ WinRM (Windows)
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ MANAGED NODE │ │ MANAGED NODE │ │ MANAGED NODE │
│   (Server 1) │ │   (Server 2) │ │   (Server 3) │
│              │ │              │ │              │
│ No Agent     │ │ No Agent     │ │ No Agent     │
│ Required!    │ │ Required!    │ │ Required!    │
└──────────────┘ └──────────────┘ └──────────────┘
```

### Components Explained:

- **Control Node**: The machine where Ansible is installed and from which automation is executed
- **Managed Nodes**: Target machines that Ansible manages (servers, VMs, containers)
- **Inventory**: List of managed nodes
- **Playbooks**: YAML files defining automation tasks
- **Modules**: Reusable units of code that perform specific tasks (e.g., `yum`, `apt`, `copy`, `service`)
- **Tasks**: Single units of action in a playbook
- **Roles**: Reusable collections of tasks, variables, and templates

---

## Important Files and Configurations

### 1. Ansible Configuration File (`ansible.cfg`)

**Location Priority** (Ansible checks in this order):
1. `ANSIBLE_CONFIG` environment variable
2. `./ansible.cfg` (current directory)
3. `~/.ansible.cfg` (home directory)
4. `/etc/ansible/ansible.cfg` (global)

**Example Configuration:**
```ini
[defaults]
# Inventory file location
inventory = ./inventory

# Remote user for SSH connections
remote_user = ansible

# SSH connection settings
host_key_checking = False
timeout = 30

# Privilege escalation
become = True
become_method = sudo
become_user = root
become_ask_pass = False

# Output settings
stdout_callback = yaml
```

### 2. Inventory File

The inventory defines which hosts Ansible will manage.

**INI Format (`inventory` or `hosts`):**
```ini
# Individual hosts
web1.example.com
192.168.1.10

# Grouped hosts
[webservers]
web1.example.com
web2.example.com
192.168.1.10

[databases]
db1.example.com
db2.example.com

# Groups of groups
[production:children]
webservers
databases

# Variables for groups
[webservers:vars]
ansible_user=ubuntu
ansible_port=22
http_port=80
```

**YAML Format (`inventory.yml`):**
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        ansible_user: ubuntu
        http_port: 80
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
```

### 3. Playbook Structure

A playbook is a YAML file containing plays (sets of tasks):

```yaml
---
- name: Playbook name
  hosts: target_hosts
  become: yes
  vars:
    variable_name: value
  
  tasks:
    - name: Task description
      module_name:
        parameter: value
```

### 4. Directory Structure (Best Practice)

```
ansible-project/
├── ansible.cfg           # Configuration file
├── inventory/            # Inventory files
│   ├── production
│   └── staging
├── playbooks/            # Playbooks
│   ├── webserver.yml
│   └── database.yml
├── roles/                # Reusable roles
│   └── nginx/
│       ├── tasks/
│       ├── handlers/
│       ├── templates/
│       ├── files/
│       └── vars/
├── group_vars/           # Variables for groups
│   ├── all.yml
│   └── webservers.yml
└── host_vars/            # Variables for specific hosts
    └── web1.yml
```

---

## Connectivity Setup

### Step 1: Install Ansible on Control Node

**On Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install ansible -y
```

**On RHEL/CentOS:**
```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

**Using pip:**
```bash
pip install ansible
```

### Step 2: Set Up SSH Key-Based Authentication

```
┌─────────────────┐                    ┌─────────────────┐
│  Control Node   │                    │  Managed Node   │
│                 │                    │                 │
│  1. Generate    │                    │                 │
│     SSH keys    │                    │                 │
│                 │                    │                 │
│  ssh-keygen     │                    │                 │
│                 │                    │                 │
│  2. Copy public │─────────────────>  │  3. Public key  │
│     key         │   ssh-copy-id      │     added to    │
│                 │                    │  ~/.ssh/        │
│                 │                    │  authorized_keys│
│                 │                    │                 │
│  4. Connect     │<──────────────────>│  5. SSH access  │
│     via Ansible │   Passwordless     │     granted     │
└─────────────────┘                    └─────────────────┘
```

**Commands:**

**On Control Node:**
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "ansible@controlnode"

# Copy public key to managed nodes
ssh-copy-id user@managed-node-ip

# Test connection
ssh user@managed-node-ip
```

### Step 3: Create Inventory File

Create `inventory` file:
```ini
[webservers]
192.168.1.10 ansible_user=ubuntu
192.168.1.11 ansible_user=ubuntu
```

### Step 4: Test Connectivity

```bash
# Ping all hosts
ansible all -i inventory -m ping

# Expected output:
# 192.168.1.10 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### Step 5: Ad-Hoc Commands (Quick Tests)

```bash
# Check uptime
ansible all -i inventory -a "uptime"

# Check disk space
ansible all -i inventory -a "df -h"

# Install package (with privilege escalation)
ansible webservers -i inventory -b -m apt -a "name=vim state=present"
```

---

## Use Case: Installing a Web Server

Let's install and configure Nginx web server on multiple nodes.

### Workflow Diagram

```
┌──────────────────────────────────────────────────────────┐
│ Step 1: Run Playbook                                     │
│ ansible-playbook -i inventory webserver.yml              │
└─────────────────────┬────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────┐
│ Step 2: Ansible connects to managed nodes via SSH       │
└─────────────────────┬────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────┐
│ Step 3: Execute tasks on each managed node              │
│  ✓ Update package cache                                 │
│  ✓ Install Nginx                                         │
│  ✓ Copy custom HTML file                                │
│  ✓ Start Nginx service                                  │
│  ✓ Enable Nginx on boot                                 │
└─────────────────────┬────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────┐
│ Step 4: Verify web server is running                    │
│ curl http://managed-node-ip                             │
└──────────────────────────────────────────────────────────┘
```

### Setup Files

**1. Create `inventory` file:**
```ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu

[webservers:vars]
ansible_become=yes
```

**2. Create `webserver.yml` playbook:**
```yaml
---
- name: Install and configure Nginx web server
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    server_name: "{{ ansible_hostname }}"
    document_root: /var/www/html

  tasks:
    - name: Update apt cache (for Debian/Ubuntu)
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install Nginx
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install Nginx (for RHEL/CentOS)
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    - name: Create custom index.html
      copy:
        content: |
          <!DOCTYPE html>
          <html>
          <head>
              <title>Welcome to {{ server_name }}</title>
              <style>
                  body {
                      font-family: Arial, sans-serif;
                      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                      display: flex;
                      justify-content: center;
                      align-items: center;
                      height: 100vh;
                      margin: 0;
                  }
                  .container {
                      background: white;
                      padding: 50px;
                      border-radius: 10px;
                      box-shadow: 0 10px 30px rgba(0,0,0,0.3);
                      text-align: center;
                  }
                  h1 { color: #667eea; }
                  p { color: #666; }
              </style>
          </head>
          <body>
              <div class="container">
                  <h1>🚀 Ansible Deployment Successful!</h1>
                  <p>Server: <strong>{{ server_name }}</strong></p>
                  <p>Deployed with Ansible automation</p>
              </div>
          </body>
          </html>
        dest: "{{ document_root }}/index.html"
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Ensure Nginx is started
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Allow HTTP traffic through firewall (UFW)
      ufw:
        rule: allow
        port: "{{ http_port }}"
        proto: tcp
      when: ansible_os_family == "Debian"
      ignore_errors: yes

    - name: Verify Nginx is responding
      uri:
        url: "http://localhost:{{ http_port }}"
        status_code: 200
      register: result
      until: result.status == 200
      retries: 5
      delay: 2

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**3. Create `ansible.cfg`:**
```ini
[defaults]
inventory = ./inventory
host_key_checking = False
stdout_callback = yaml
```

### Execution

**Run the playbook:**
```bash
ansible-playbook webserver.yml
```

**Expected output:**
```
PLAY [Install and configure Nginx web server] *********************************

TASK [Gathering Facts] *********************************************************
ok: [web1]
ok: [web2]

TASK [Update apt cache (for Debian/Ubuntu)] ************************************
changed: [web1]
changed: [web2]

TASK [Install Nginx] ***********************************************************
changed: [web1]
changed: [web2]

TASK [Create custom index.html] ************************************************
changed: [web1]
changed: [web2]

TASK [Ensure Nginx is started] *************************************************
changed: [web1]
changed: [web2]

PLAY RECAP *********************************************************************
web1                       : ok=5    changed=4    unreachable=0    failed=0
web2                       : ok=5    changed=4    unreachable=0    failed=0
```

### Verification

**Test the web servers:**
```bash
# From control node
curl http://192.168.1.10
curl http://192.168.1.11

# Or use Ansible
ansible webservers -m uri -a "url=http://localhost return_content=yes"
```

### Advanced: Using Roles

Create a reusable role structure:

```bash
ansible-galaxy init nginx
```

**Directory structure:**
```
nginx/
├── tasks/
│   └── main.yml        # Installation and configuration tasks
├── handlers/
│   └── main.yml        # Service restart handlers
├── templates/
│   └── index.html.j2   # Jinja2 template for dynamic content
├── files/
│   └── nginx.conf      # Static files to copy
├── vars/
│   └── main.yml        # Role variables
└── defaults/
    └── main.yml        # Default variables
```

**Use the role in a playbook:**
```yaml
---
- name: Deploy web servers using role
  hosts: webservers
  become: yes
  roles:
    - nginx
```

---

## Common Ansible Modules

| Module | Purpose | Example |
|--------|---------|---------|
| `ping` | Test connectivity | `ansible all -m ping` |
| `apt/yum` | Package management | `apt: name=nginx state=present` |
| `copy` | Copy files | `copy: src=file.txt dest=/tmp/` |
| `template` | Copy Jinja2 templates | `template: src=config.j2 dest=/etc/app/config` |
| `service` | Manage services | `service: name=nginx state=started` |
| `user` | Manage users | `user: name=johndoe state=present` |
| `file` | Manage files/directories | `file: path=/tmp/test state=directory` |
| `git` | Clone repositories | `git: repo=https://github.com/... dest=/opt/app` |
| `shell/command` | Run commands | `shell: echo "Hello" > /tmp/hello.txt` |
| `uri` | HTTP requests | `uri: url=http://example.com` |

---

## Best Practices

1. **Use version control**: Store playbooks in Git
2. **Idempotency**: Write playbooks that can run multiple times safely
3. **Use roles**: Organize complex playbooks into reusable roles
4. **Variables**: Use group_vars and host_vars for environment-specific configs
5. **Vault**: Encrypt sensitive data with `ansible-vault`
6. **Tags**: Use tags to run specific parts of playbooks
7. **Check mode**: Test with `--check` flag before applying changes
8. **Documentation**: Comment your playbooks and maintain README files

---

## Quick Reference Commands

```bash
# Run a playbook
ansible-playbook playbook.yml

# Check syntax
ansible-playbook playbook.yml --syntax-check

# Dry run (no changes)
ansible-playbook playbook.yml --check

# Run specific tags
ansible-playbook playbook.yml --tags "install,configure"

# Limit to specific hosts
ansible-playbook playbook.yml --limit web1

# Verbose output
ansible-playbook playbook.yml -v    # -vv, -vvv for more verbosity

# List hosts
ansible all --list-hosts

# Ad-hoc command
ansible webservers -m shell -a "uptime"
```

---

## Conclusion

You now have a solid foundation in Ansible! This tutorial covered:
- ✅ Ansible architecture and components
- ✅ Important configuration files
- ✅ SSH connectivity setup
- ✅ Practical web server deployment

**Next Steps:**
- Explore Ansible Galaxy for community roles
- Learn about Ansible Vault for secrets management
- Study advanced topics like dynamic inventory and custom modules
- Practice with more complex orchestration scenarios

Happy automating! 🚀
