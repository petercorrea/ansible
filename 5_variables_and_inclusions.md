# 5. Managing Variables and Inclusions

In Ansible, variables and inclusions are powerful features that allow you to create more dynamic and reusable playbooks. Variables can be defined in various places within Ansible, including in playbooks, in inventory, in separate files, or passed at the command line. Inclusions allow you to include tasks, handlers, or even entire playbooks within other playbooks, facilitating code reuse and modularization.

In Ansible, understanding variable precedence is crucial for effectively managing and utilizing variables across your playbooks, roles, inventory, and command-line invocations. Additionally, leveraging facts—system-specific information gathered by Ansible—can significantly enhance playbook logic and decision-making.

## Variables

#### Defining Variables

Variables in Ansible can be used to store values that you want to use in your playbooks. They can be defined in several places:

**Playbooks**: Directly under the vars section.

**Inventory Files**: Alongside hosts or groups for specific configurations.

**Separate Files**: Usually under a vars directory for organization, which can then be included in playbooks as needed.

**Command Line**: Using the -e or --extra-vars option with ansible-playbook command.

Example of defining variables in a playbook:

```yaml
---
- name: Install and start Apache
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  tasks:
    ...
```

#### Variable Precedence Explained with Examples

Ansible resolves variables by following a specific precedence. Here's a breakdown with examples for each level:

1. **Command Line Values (`-e` or `--extra-vars`)**:

   ```bash
   ansible-playbook playbook.yml -e "http_port=8080"
   ```

   Overrides any `http_port` variable defined elsewhere.

2. **Role Defaults** (defined in `roles/x/defaults/main.yml`):

   ```yaml
   # roles/webserver/defaults/main.yml
   http_port: 80
   ```

   The most "default" level of variables, easily overridden.

3. **Inventory Vars**:

   ```ini
   # inventory.ini
   [webservers]
   web1.example.com http_port=8080
   ```

   Specific to hosts, can override role defaults.

4. **Inventory `group_vars`**:

   ```yaml
   # group_vars/webservers.yml
   http_port: 8080
   ```

   Applies to all members of the `webservers` group.

5. **Inventory `host_vars`**:

   ```yaml
   # host_vars/web1.example.com.yml
   http_port: 8081
   ```

   Specific to `web1.example.com`, overrides group_vars.

6. **Playbook `group_vars`**:

   ```yaml
   # Within playbook directory structure
   http_port: 8082
   ```

   If defined in a playbook's adjacent `group_vars` directory.

7. **Playbook `host_vars`**:

   ```yaml
   # Within playbook directory structure
   http_port: 8083
   ```

   Similar to playbook `group_vars` but for specific hosts.

8. **Host Facts**:
   Facts are gathered by default and contain system-specific information. Not directly set by users but can influence variables through conditional logic.

9. **Play Vars** (vars defined in a play):

   ```yaml
   # In playbook.yml
   vars:
     http_port: 8084
   ```

   Directly in a play within a playbook.

10. **Play `vars_prompt`**:
    Interactive variables prompted at playbook run time.

11. **Play `vars_files`**:

    ```yaml
    # In playbook.yml
    vars_files:
      - vars/main.yml
    ```

    Variables loaded from files listed under `vars_files`.

12. **Role Vars** (defined in `roles/x/vars/main.yml`):
    Hardcoded, safest place to define variables that should not be overridden.

13. **Block Vars** (variables scoped to a block of tasks):

    ```yaml
    tasks:
    - block:
        - debug:
            msg: "This is a block variable"
      vars:
        block_var: value
    ```

14. **Task Vars** (variables defined for a specific task):

    ```yaml
    - name: Print message
      debug:
        msg: "Task variable example"
      vars:
        task_var: value
    ```

15. **Include Vars** (`include_vars` module):

    ```yaml
    - name: Include variables
      include_vars: 'file.yml'
    ```

16. **Set Facts / Registered Vars** (`set_fact` module):

    ```yaml
    - name: Set a fact
      set_fact:
        fact_name: value
    ```

17. **Role (and `include_role`) Params**:
    Passed as parameters to roles or included roles.

18. **Include Params**:
    Variables passed to includes.

### Magic Variables

Magic variables in Ansible are special variables that provide access to data about the play, the current host, and other aspects of the execution environment. These variables can be incredibly useful for dynamic playbook behavior, accessing system information, and debugging. Here's an overview of some of the key magic variables and how they are used:

#### 1. `hostvars`

- **Description**: Contains information about all the hosts in the inventory, making it possible to access variables related to a different host than the one currently being managed.
- **Example Usage**: Accessing a variable from another host.

  ```yaml
  - debug:
      msg: "{{ hostvars['otherhost']['ansible_facts']['os_family'] }}"
  ```

#### 2. `group_names`

- **Description**: A list of all the groups the current host is a part of. This can be useful for applying certain configurations based on the group membership of a host.
- **Example Usage**: Conditional tasks based on group membership.

  ```yaml
  - name: Apply this task to hosts in the 'webservers' group only
    debug:
      msg: "This host is a webserver."
    when: "'webservers' in group_names"
  ```

#### 3. `groups`

- **Description**: Contains a map of all the groups in the inventory and each group's list of member hosts. It can be used for operations that need to consider the entire inventory structure.
- **Example Usage**: Iterating over all hosts in a specific group.

  ```yaml
  - name: Print all hosts in the 'dbservers' group
    debug:
      msg: "{{ item }}"
    loop: "{{ groups['dbservers'] }}"
  ```

#### 4. `inventory_hostname`

- **Description**: The hostname of the current host, as specified in the inventory. This variable is often used to get the specific configuration for a host.
- **Example Usage**: Accessing host-specific variables.

  ```yaml
  - debug:
      msg: "Working on host {{ inventory_hostname }}"
  ```

#### 5. `inventory_hostname_short`

- **Description**: The short version of `inventory_hostname`, usually up to the first period, which can be useful for templates or when a shorter hostname is required.
- **Example Usage**: Using short hostname in tasks.

  ```yaml
  - debug:
      msg: "Short hostname is {{ inventory_hostname_short }}"
  ```

#### 6. `ansible_facts`

- **Description**: Contains all the gathered facts about the current host. This variable is a dictionary of system-specific information collected by Ansible's fact gathering.
- **Example Usage**: Accessing system facts.

  ```yaml
  - debug:
      msg: "This host uses {{ ansible_facts['os_family'] }} family."
  ```

#### 7. `play_hosts`

- **Description**: A list of all the hosts that are targeted in the current play. This can be useful for operations that need to be aware of all the hosts being managed in a single play.
- **Example Usage**: Iterating over all hosts in the play.

  ```yaml
  - name: Print names of all hosts in the play
    debug:
      msg: "{{ item }}"
    loop: "{{ play_hosts }}"
  ```

## Facts

Facts are system and environment information collected by Ansible from managed nodes. They can be used in playbooks to make decisions based on actual system properties.

### List of Commonly Available Facts

Ansible gathers a wealth of system-specific information, known as facts, which you can use in your playbooks. Here's a list of commonly used facts:

- `ansible_distribution`: The distribution of the managed host (e.g., Ubuntu, CentOS).
- `ansible_os_family`: The OS family (e.g., Debian, RedHat).
- `ansible_architecture`: The architecture of the managed host (e.g., x86_64).
- `ansible_facts['network_interfaces']`: Information about the host's network interfaces.
- `ansible_facts['memtotal_mb']`: Total memory in MB.
- `ansible_facts['processor_cores']`: Number of processor cores.
- `ansible_facts['ipv4']`: IPv4 addresses of the host.

You can display all facts about a host using the `setup`

 module:

```yaml
- name: Gather facts
  hosts: all
  tasks:
    - name: Display all facts
      ansible.builtin.setup:
```

## Including Files and Roles

**Includes**: Allow you to include tasks or variables from external files.

**include_tasks**: Dynamically includes a task list.

**import_tasks**: Static import of a task list.

**Roles**: Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Gathering your content into roles allows you to reuse code and simplify playbook creation.

Example of using roles:

```yaml
---
- name: Install and configure web server
  hosts: webservers
  roles:
    - common
    - webserver
```

This setup assumes you have a directory structure like:

```text
roles/
  common/
    tasks/
      main.yml
  webserver/
    tasks/
      main.yml
```

Where each main.yml under tasks directory would contain the relevant tasks for the role.

### Best Practices for Variables and Inclusions

Keep your variables organized and documented. Use comments in your YAML files to explain the purpose of each variable.
Use roles for common sets of tasks and variables to promote reuse across your playbooks.
Leverage group_vars and host_vars to manage environment-specific configurations.

## Register

In Ansible, the `register` keyword is used to capture the output of a task. This allows you to store the results of a command or module execution in a variable, which can then be used in subsequent tasks for conditional execution, debugging, or any other purpose where the output of one task needs to influence another.

### Basic Usage of `register`

Here's a simple example that demonstrates how to use `register` to capture the output of a task:

```yaml
---
- name: Register example playbook
  hosts: all
  tasks:
    - name: Check disk usage
      ansible.builtin.command: df -h
      register: disk_usage

    - name: Print disk usage
      ansible.builtin.debug:
        var: disk_usage.stdout
```

In this example:

- The `ansible.builtin.command` module is used to execute the `df -h` command on the remote host(s).
- The output of the command is registered (captured) in the variable `disk_usage`.
- The `ansible.builtin.debug` module then prints the standard output (`stdout`) of the command to the console.

### Conditional Execution Based on Registered Variables

Registered variables can also be used to conditionally execute tasks based on the output of previous tasks.

```yaml
---
- name: Conditional execution with register
  hosts: all
  tasks:
    - name: Check if nginx is installed
      ansible.builtin.command: which nginx
      register: nginx_installed
      ignore_errors: true

    - name: Install nginx if not already installed
      ansible.builtin.yum:
        name: nginx
        state: present
      when: nginx_installed.rc != 0
```

In this playbook:

- The `which nginx` command checks if nginx is installed on the remote host(s).
- The `ignore_errors: true` option is used to ensure the playbook continues executing even if the command indicates that nginx is not installed (which results in a non-zero return code).
- The `register` keyword captures the output of the `which` command in the `nginx_installed` variable.
- The `when` condition checks the return code (`rc`) of the registered variable. If the return code is not `0` (indicating nginx was not found), the subsequent task to install nginx is executed.

### Advanced Use: Looping and Registered Variables

You can use `register` in conjunction with loops to capture the output of tasks executed multiple times. Ansible automatically converts the registered variable into a list of results for each item in the loop.

```yaml
---
- name: Register with loop example
  hosts: all
  tasks:
    - name: Check status of multiple services
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
      loop:
        - httpd
        - nginx
      register: service_status

    - name: Print service status
      ansible.builtin.debug:
        msg: "Service {{ item.item }} status: {{ item.stdout }}"
      loop: "{{ service_status.results }}"
      when: item.stdout is defined
```

In this scenario:

- A loop is used with the `ansible.builtin.service` module to check the status of multiple services (`httpd` and `nginx`).
- The output for each iteration of the loop is captured in the `service_status` variable.
- The `service_status.results` list contains the output for each loop iteration, which can be iterated over in a subsequent task.
- The `ansible.builtin.debug` task iterates over each item in `service_status.results`, printing the status of each service.

The `register` keyword is a powerful feature in Ansible that enhances playbook flexibility and allows for dynamic responses based on the state of your managed nodes.
