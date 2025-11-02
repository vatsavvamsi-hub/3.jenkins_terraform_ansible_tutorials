# Ansible Basics Tutorial

## What is Ansible?

Ansible is an open-source automation tool that allows you to manage infrastructure, deploy applications, and perform IT tasks across multiple machines. It uses agentless architecture and works over SSH.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                  ANSIBLE CONTROL NODE                    │
│         (Where Ansible is installed)                     │
│                                                           │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Playbooks | Inventory | Roles | Modules         │  │
│  └────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
         │                    │                    │
         │ SSH                │ SSH                │ SSH
         │                    │                    │
         ▼                    ▼                    ▼
    ┌─────────┐          ┌─────────┐          ┌─────────┐
    │ Server1 │          │ Server2 │          │ Server3 │
    │(Managed)│          │(Managed)│          │(Managed)│
    └─────────┘          └─────────┘          └─────────┘
```

---

## Core Concepts

### 1. **Inventory**
Defines the hosts you want to manage.

```ini
# Static Inventory (inventory.ini)

[webservers]
web1.example.com
web2.example.com
192.168.1.10

[databases]
db1.example.com
db2.example.com

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 2. **Playbooks**
YAML files containing a set of tasks to execute on hosts.

```yaml
# Example Playbook (site.yml)
---
- name: Configure web servers
  hosts: webservers
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      
    - name: Install nginx
      apt:
        name: nginx
        state: present
      
    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes
```

### 3. **Modules**
Pre-built units of code that perform specific tasks (apt, service, file, copy, etc.)

```
┌──────────────────────────────────────────┐
│           Common Modules                  │
├──────────────────────────────────────────┤
│ • apt/yum         - Package management   │
│ • service         - Service management   │
│ • file            - File management      │
│ • copy            - Copy files           │
│ • template        - Template rendering   │
│ • user/group      - User/group mgmt      │
│ • shell/command   - Execute commands     │
│ • debug           - Print variables      │
└──────────────────────────────────────────┘
```

### 4. **Hosts & Groups**
Organized collections of managed nodes.

---

## Playbook Execution Flow

```
START
  │
  ▼
┌──────────────────────────┐
│ Load Playbook (YAML)     │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ Parse Inventory          │
│ Identify Target Hosts    │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ For Each Host:           │
│ • Connect via SSH        │
│ • Transfer Task Code     │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ Execute Tasks Sequentially
│ (Task 1, Task 2, ...)    │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ Gather Results           │
│ Report Status            │
└──────────┬───────────────┘
           │
           ▼
         END
```

---

## Getting Started - Step by Step

### Step 1: Install Ansible (on Control Node)

```bash
# On Ubuntu/Debian
sudo apt update
sudo apt install ansible

# On RHEL/CentOS
sudo yum install ansible

# On macOS
brew install ansible

# Verify installation
ansible --version
```

### Step 2: Set Up SSH Keys

```bash
# Generate SSH key pair (if not exists)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key

# Copy public key to managed hosts
ssh-copy-id -i ~/.ssh/ansible_key.pub user@host1
ssh-copy-id -i ~/.ssh/ansible_key.pub user@host2
```

### Step 3: Create Inventory File

```ini
# ~/.ansible/hosts or /etc/ansible/hosts

[webservers]
web1.example.com ansible_user=ubuntu
web2.example.com ansible_user=ubuntu

[databases]
db1.example.com ansible_user=ubuntu

[all:vars]
ansible_ssh_private_key_file=~/.ssh/ansible_key
```

### Step 4: Test Connectivity

```bash
# Ping all hosts
ansible all -i inventory.ini -m ping

# Expected output:
# web1.example.com | SUCCESS => {"ping": "pong"}
# web2.example.com | SUCCESS => {"ping": "pong"}
# db1.example.com | SUCCESS => {"ping": "pong"}
```

---

## Common Commands

```bash
# Run adhoc command on all hosts
ansible all -m command -a "uptime"

# Run against specific group
ansible webservers -m shell -a "systemctl status nginx"

# Run playbook
ansible-playbook site.yml

# Run playbook with specific hosts
ansible-playbook site.yml -i inventory.ini --limit webservers

# Dry-run (check mode) - show what would change
ansible-playbook site.yml --check

# Verbose output
ansible-playbook site.yml -v  # or -vv, -vvv for more detail

# List all hosts
ansible all --list-hosts

# Get host facts
ansible web1.example.com -m setup
```

---

## Basic Playbook Examples

### Example 1: Install Package and Start Service

```yaml
---
- name: Deploy NGINX
  hosts: webservers
  become: yes  # Run with sudo
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Enable nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

### Example 2: Copy Config File and Restart Service

```yaml
---
- name: Update and restart nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Copy nginx config
      copy:
        src: ./nginx.conf
        dest: /etc/nginx/nginx.conf
        backup: yes
    
    - name: Validate nginx config
      shell: nginx -t
    
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### Example 3: User Management

```yaml
---
- name: Manage users
  hosts: all
  become: yes
  tasks:
    - name: Create user
      user:
        name: appuser
        state: present
        shell: /bin/bash
        home: /home/appuser
    
    - name: Add user to sudo group
      user:
        name: appuser
        groups: sudo
        append: yes
```

---

## Variables & Facts

### Using Variables

```yaml
---
- name: Variables example
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  tasks:
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - curl
        - git
    
    - name: Set configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
```

### Gathering Facts

```yaml
---
- name: Show system facts
  hosts: all
  tasks:
    - name: Display OS
      debug:
        msg: "Operating System: {{ ansible_distribution }}"
    
    - name: Display IP address
      debug:
        msg: "IP Address: {{ ansible_default_ipv4.address }}"
```

---

## Conditional Execution

```yaml
---
- name: Conditionals example
  hosts: all
  tasks:
    - name: Install Apache on Debian-based systems
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install Apache on RHEL-based systems
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
```

---

## Loops & Iteration

```yaml
---
- name: Loops example
  hosts: all
  tasks:
    - name: Create multiple files
      file:
        path: "/tmp/file_{{ item }}.txt"
        state: touch
      loop:
        - one
        - two
        - three
    
    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - curl
        - wget
        - git
        - vim
```

---

## Roles - Organizing Complex Playbooks

```
roles/
├── webserver/
│   ├── tasks/
│   │   └── main.yml
│   ├── handlers/
│   │   └── main.yml
│   ├── templates/
│   │   └── nginx.conf.j2
│   ├── files/
│   └── vars/
│       └── main.yml
└── database/
    ├── tasks/
    │   └── main.yml
    └── vars/
        └── main.yml
```

### Using Roles in Playbook

```yaml
---
- name: Deploy complete stack
  hosts: all
  roles:
    - webserver
    - database
    - monitoring
```

---

## Best Practices

```
┌─────────────────────────────────────────────────┐
│          Ansible Best Practices                 │
├─────────────────────────────────────────────────┤
│ 1. Use roles to organize code                   │
│ 2. Use variables for flexibility                │
│ 3. Use tags for selective execution             │
│ 4. Use handlers for service restarts            │
│ 5. Maintain idempotent tasks                    │
│ 6. Use descriptive task names                   │
│ 7. Test playbooks with --check mode             │
│ 8. Use version control (Git)                    │
│ 9. Avoid hardcoding values                      │
│ 10. Use meaningful inventory organization       │
└─────────────────────────────────────────────────┘
```

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| SSH Connection refused | Check SSH key permissions (600), check firewall |
| Permission denied | Use `become: yes` for sudo, check user permissions |
| Module not found | Install required package on managed node |
| Task fails silently | Use `-v` or `-vvv` flag for verbose output |
| Timeout | Increase timeout, check network connectivity |

### Debug Tasks

```yaml
---
- name: Debugging
  hosts: all
  tasks:
    - name: Print variable
      debug:
        var: ansible_os_family
    
    - name: Print custom message
      debug:
        msg: "Host: {{ inventory_hostname }}, IP: {{ ansible_default_ipv4.address }}"
    
    - name: Register command output
      shell: uptime
      register: uptime_result
    
    - name: Print registered output
      debug:
        var: uptime_result.stdout
```

---

## Quick Reference Diagram

```
ANSIBLE WORKFLOW
═════════════════════════════════════════════════════════════

┌─────────────────────┐
│  Write Playbook     │
│   (YAML file)       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Run Playbook       │
│  ansible-playbook   │
│     site.yml        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Load Inventory     │
│  (hosts list)       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  SSH to each host   │
│  Transfer + Execute │
│  Python scripts     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Gather & Report    │
│  Results            │
└──────────┬──────────┘
           │
           ▼
      SUCCESS ✓
```

---

## Next Steps

1. **Install Ansible** on your control machine
2. **Set up SSH keys** to your managed hosts
3. **Create an inventory** file with your hosts
4. **Write your first playbook** for a simple task
5. **Run and test** with `--check` mode first
6. **Organize with roles** as complexity grows
7. **Use version control** to track your playbooks

---

## Useful Resources

- **Official Docs**: https://docs.ansible.com/
- **Module Index**: https://docs.ansible.com/ansible/latest/modules/
- **Galaxy (Pre-built Roles)**: https://galaxy.ansible.com/
- **Best Practices**: https://docs.ansible.com/ansible/latest/tips_tricks/

---

## Summary

**Ansible** provides:
- ✅ Agentless automation
- ✅ Simple YAML syntax
- ✅ SSH-based security
- ✅ Powerful orchestration
- ✅ Scalable infrastructure management
- ✅ Idempotent operations (safe to run repeatedly)

Start small, test with `--check`, and scale up!
