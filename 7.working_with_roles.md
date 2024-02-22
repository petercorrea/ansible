### 7. Working with Roles

Ansible roles are a way to organize playbooks and allow for reuse of playbook code. They provide a framework for fully independent, or interdependent collections of variables, tasks, files, templates, and modules. In essence, roles are units of organization in Ansible, making your automation code more modular and reusable.

#### Roles Overview

A role enables the sharing of automation with others by bundling automation content and making it easier to transfer automation across different environments. Roles are structured in a specific way, allowing Ansible to automatically load certain `vars_files`, `tasks`, and `handlers` based on a known file structure.

#### Creating and Using Roles

**Creating a Role:**
You can create a new role using the `ansible-galaxy` command, which is part of Ansible. For example, to create a role named `webserver`, you would run:

```bash
ansible-galaxy init webserver
```

This command creates a directory structure for the role in your current directory:

```
webserver/
    defaults/    # Default variables for the role
    handlers/    # Handlers, which can be used by this role or even anywhere outside this role
    meta/        # Metadata for the role, including dependencies
    README.md    # A markdown file with instructions on the role
    tasks/       # The main list of tasks that the role executes
    templates/   # Templates files, which can be deployed via this role
    tests/       # Test files for the role
    vars/        # Other variables for the role, typically with higher precedence than defaults
```

**Using Roles:**
To use a role, you include it in your playbook at the appropriate point. For example:

```yaml
---
- name: Deploy webserver
  hosts: webservers
  roles:
    - webserver
```

This playbook applies the `webserver` role to all hosts in the `webservers` group.

#### Role Directory Structure

- **tasks/main.yml**: The main set of tasks that the role executes.
- **handlers/main.yml**: Handlers, which tasks can notify to take action when certain conditions are met.
- **defaults/main.yml**: Default variables for the role. These have the lowest precedence.
- **vars/main.yml**: Other variables for the role, typically with higher precedence than defaults.
- **files/**: Contains files that can be deployed via this role.
- **templates/**: Contains templates that can be deployed via this role.
- **meta/main.yml**: Defines some metadata for the role, including the role's dependencies.

#### Using Roles from Ansible Galaxy

Ansible Galaxy is Ansible's official hub for sharing Ansible content. You can download roles from Ansible Galaxy to use in your playbooks, which can save time and effort in writing complex automation scripts.

To install a role from Ansible Galaxy, use the `ansible-galaxy` command:

```bash
ansible-galaxy install username.rolename
```

And then include it in your playbook as shown earlier.

#### Best Practices for Roles

- **Modularity**: Keep roles small and focused. Each role should have a clear, single responsibility.
- **Reusability**: Design roles to be reusable in different environments. This often means making roles configurable via role variables.
- **Encapsulation**: Use role defaults and variables to encapsulate the role's configuration, minimizing dependencies on global playbook variables.
