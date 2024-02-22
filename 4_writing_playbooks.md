# 4. Writing Ansible Playbooks

#### YAML Syntax

YAML (YAML Ain't Markup Language) is a human-readable data serialization standard that Ansible uses for its playbook syntax. Understanding YAML syntax is crucial for writing effective Ansible playbooks.

- YAML is case sensitive.
- Uses indentation to represent hierarchy (typically 2 spaces per indent level).
- Lists are created using a dash (`-`) followed by a space.
- Dictionaries (key-value pairs) are represented by `key: value` pairs.

Example of YAML syntax:

```yaml
---
- hosts: webservers
  vars:
    http_port: 80
  tasks:
  - name: Ensure nginx is installed
    ansible.builtin.yum:
      name: nginx
      state: present
```

#### Basic Playbook Structure

A playbook is a list of plays. A play is a mapping between a set of hosts and the tasks that should be run on those hosts.

- **hosts**: Specifies the hosts or group of hosts upon which commands, modules, and tasks in a playbook operate.
- **vars**: Defines variables for use in the playbook.
- **tasks**: Lists the tasks to be executed.

Example playbook:

```yaml
---
- name: Configure webserver with nginx
  hosts: webservers
  become: yes  # Use privilege escalation
  vars:
    server_port: 80
  tasks:
    - name: Install nginx
      ansible.builtin.yum:
        name: nginx
        state: latest

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes
```

#### Tasks, Handlers, and Tags

- **Tasks**: The most fundamental block of a playbook, defining what action to perform on the host.
- **Handlers**: Special tasks that only run when notified by another task. Typically used for restarting services.
- **Tags**: Allow you to run specific parts of a playbook without running the whole playbook.

Example of tasks and handlers:

```yaml
---
- name: Configure webserver
  hosts: webservers
  tasks:
    - name: Install nginx
      ansible.builtin.yum:
        name: nginx
        state: latest
      notify: restart nginx

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
        enabled: yes
```

#### Variables and Facts

Variables in Ansible are used to manage differences between systems. Facts are a type of variable that Ansible gathers from the managed hosts.

Example using variables:

```yaml
---
- name: Install and start Apache
  hosts: webservers
  vars:
    http_port: 80
  tasks:
    - name: Install apache
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start apache service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
      when: ansible_facts['distribution'] == 'CentOS'
```

#### Loops and Conditionals

Loops allow you to repeat a task for each item in a list. Conditionals let you execute tasks based on conditions.

Example of a loop:

```yaml
---
- name: Add several users
  hosts: all
  tasks:
    - name: Add users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
        groups: "wheel"
      loop:
        - alice
        - bob
```

Example of conditionals:

```yaml
---
- name: Start a service on a particular distribution
  hosts: all
  tasks:
    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
      when: ansible_facts['os_family'] == "Debian"
```
