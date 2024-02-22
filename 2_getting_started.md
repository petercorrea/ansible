# 2. Getting Started with Ansible

#### Installation and Configuration

**Installation**:

- Ansible can be installed on various platforms. For Linux distributions like Ubuntu, it can be installed via apt, for Red Hat-based systems via yum, and for macOS using brew. Ansible can also be run from any machine with Python installed, including Windows (via WSL or a virtual environment).

```bash
# For Ubuntu/Debian systems
sudo apt update
sudo apt install ansible

# For Red Hat/CentOS systems
sudo yum install ansible

# For macOS
brew install ansible
```

**Configuration**:

- After installation, Ansible looks for its configuration settings in several places: `ansible.cfg` in the current directory, `.ansible.cfg` in the user's home directory, or `/etc/ansible/ansible.cfg`.
- Key settings you might want to customize include the default inventory file location, remote user, and privilege escalation settings.

#### Ansible Inventory Basics

The inventory is a list of nodes or hosts (with their IP addresses or domain names) that Ansible manages. It can be a simple static file in INI or YAML format or a dynamic source.

- **Static Inventory**:
  - **INI Format**: A simple format where hosts are listed under group names.
  - **YAML Format**: Offers more flexibility and readability, especially for complex configurations.

```ini
# INI format example
[webservers]
webserver1.example.com
webserver2.example.com

[dbservers]
dbserver.example.com
```

```yaml
# YAML format example
all:
  children:
    webservers:
      hosts:
        webserver1.example.com:
        webserver2.example.com:
    dbservers:
      hosts:
        dbserver.example.com:
```

- **Groups and Host Variables**: You can define variables at the group level or for individual hosts, which allows for more flexible configurations.

#### Managing Ansible Configurations and Inventory

- **ansible.cfg Customization**: This file can be used to set defaults for many aspects of Ansible's operation. Common customizations include specifying a default inventory file, adjusting the number of parallel tasks, and setting up host key checking.

- **Dynamic Inventory**: For environments where hosts change frequently, Ansible supports dynamic inventories. Instead of a static file, a script can generate the inventory from external data sources (e.g., cloud providers).

```bash
# Example of specifying a custom inventory file in ansible.cfg
[defaults]
inventory = ./path/to/custom/inventory.yaml
```

#### First Steps with Ad-Hoc Commands

Ad-hoc commands are a quick way to perform tasks without writing a playbook. They are great for tasks you need to do quickly and only once.

- **Syntax**: `ansible [pattern] -m [module] -a "[module options]"`

```bash
# Ping all hosts
ansible all -m ping

# Check disk usage on all database servers
ansible dbservers -m shell -a "df -h"
```

These ad-hoc commands are powerful for quick operations across your managed nodes, offering an immediate way to perform actions without the overhead of writing a playbook.
