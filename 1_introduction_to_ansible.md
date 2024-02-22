# 1. Introduction to Ansible

#### What is Ansible?

- Ansible is an open-source automation tool, or platform, used for IT tasks such as configuration management, application deployment, intra-service orchestration, and provisioning. It aims to provide large productivity gains to a wide variety of automation challenges.

#### Why Ansible?

- **Simple to Use**: It uses a simple syntax (YAML) for its playbooks.
- **Agentless**: There’s no need to install any agent software on the nodes it manages. It uses SSH for communication.
- **Powerful & Flexible**: Ansible has a vast collection of built-in modules, supporting a wide variety of tasks.
- **Efficient**: Because it’s agentless, it reduces the overhead on your network by preventing the need for a continuous connection to managed nodes.

#### Ansible Architecture

- **Control Node**: The machine where Ansible is installed and from which all tasks and playbooks are run.
- **Managed Nodes**: The servers (nodes) being managed by Ansible.
- **Inventory**: A list of nodes being managed by Ansible.
- **Modules**: The units of code Ansible executes. Ansible executes a module by sending it to all the nodes in the inventory.
- **Playbooks**: YAML files that describe the desired state of your systems, which are used to guide Ansible's execution.
- **Facts**: Information derived about the state of the systems.

#### Key Components

- **Modules**: These are the workhorses of Ansible, which perform the actual tasks.
- **Playbooks**: The primary configuration, deployment, and orchestration language of Ansible, they allow for the launch of tasks sequentially and selectionally against custom-defined data sets.
- **Inventory**: It can be a simple file defining hosts or can dynamically pull host information from cloud services or other sources.
- **Roles**: Organize playbooks into reusable files to manage aspects of system configuration.

### Getting Started

Before diving into practical examples, ensure Ansible is installed on your system. On most Linux distributions, you can install Ansible via the package manager.

For example, on Ubuntu-based systems, you can install Ansible with:

```bash
sudo apt update
sudo apt install ansible
```

Verify the installation with:

```bash
ansible --version
```

This command should return the version of Ansible installed.

Next, let's create a simple inventory file to manage our nodes. Create a file named `hosts` with the following content:

```ini
[web]
server1.example.com

[db]
server2.example.com
```

This file defines two groups, `web` and `db`, each containing a server. Replace `server1.example.com` and `server2.example.com` with the hostnames or IP addresses of your servers.

You can now run an Ansible ad-hoc command to ping all servers in your inventory:

```bash
ansible all -i hosts -m ping
```

This command uses the `ping` module to check the connection to all hosts defined in the `hosts` file.
