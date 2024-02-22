### 6. Task Control in Ansible

Task control in Ansible involves managing the execution flow of tasks within your playbooks. This includes using handlers for service control, conditional execution of tasks, error handling, and loops for repeated execution of tasks. Understanding these concepts allows you to create more dynamic and responsive playbooks.

#### Using Handlers for Service Control

**Handlers** are special tasks that only run when they are notified by another task. They are commonly used for restarting services when configuration files change.

Example playbook with a handler:

```yaml
---
- name: Configure web server
  hosts: webservers
  tasks:
    - name: Install nginx
      ansible.builtin.yum:
        name: nginx
        state: present
      notify: restart nginx

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

In this example, the "restart nginx" handler is triggered by the "Install nginx" task if it makes changes (e.g., installing nginx).

#### Conditional Execution of Tasks

Conditionals allow you to execute tasks based on specified conditions using the `when` statement. This can depend on the value of a variable, the result of a previous task, or the presence of a certain fact.

Example of conditional task execution:

```yaml
---
- name: Conditional task example
  hosts: all
  tasks:
    - name: Install apache on CentOS
      ansible.builtin.yum:
        name: httpd
        state: present
      when: ansible_facts['os_family'] == "RedHat"
```

This task installs Apache only on hosts identified as part of the RedHat OS family.

#### Error Handling

Ansible provides ways to handle errors during playbook execution. The `ignore_errors` directive allows a task to fail without stopping the entire playbook. The `block` and `rescue` sections can be used to define error handling logic.

Example of error handling with `block`, `rescue`, and `always`:

```yaml
---
- name: Error handling example
  hosts: webservers
  tasks:
    - block:
        - name: Attempt risky operation
          command: /bin/false
      rescue:
        - name: Handle the error
          debug:
            msg: "An error occurred."
      always:
        - name: Always do this
          debug:
            msg: "This task always runs."
```

This setup attempts a risky operation, handles errors if the operation fails, and runs a task that always executes, regardless of previous success or failure.

#### Loops

Loops in Ansible allow you to repeat a task for each item in a list or result set. The `loop` keyword is used to iterate over a provided list.

Example of using loops:

```yaml
---
- name: Add several users
  hosts: all
  tasks:
    - name: Add user
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
        groups: "wheel"
      loop:
        - alice
        - bob
        - charlie
```

This playbook adds several users, iterating over each item in the list provided to the `loop` keyword.
