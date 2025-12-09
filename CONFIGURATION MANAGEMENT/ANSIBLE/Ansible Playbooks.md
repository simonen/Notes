
An Ansible playbook is structured around **plays**, and each play is a collection of key elements that define the hosts, tasks, and configuration settings. Here’s a breakdown of the main keys commonly used in an Ansible playbook.

play = item

#### Key Items

`hosts`

- **Purpose**: Specifies which hosts or group of hosts the play should target, as defined in the Ansible inventory.
- **Location**: Play level (mandatory).
- **Example**

``` yaml
- hosts: webservers
```

#### 1. `name`

- **Purpose**: Provides a description for the play or a specific task.
- **Location**: Found at the play level and task level.
- **Example**

``` yaml
- name: Configure Web Servers
```


`tasks`

- **Purpose**: Defines the list of actions (tasks) to be executed on the specified hosts.
- **Location**: Play level.
- **Example**

``` yaml
tasks:
  - name: Install Nginx
    apt:
      name: nginx
      state: present
```

`become`

- **Purpose**: Enables privilege escalation (e.g., sudo) if set to `true`.
- **Location**: Play level or task level.
- **Example**:

``` yaml
- hosts: webservers
  become: true
```

5. `gather_facts`

- **Purpose**: Controls whether Ansible should collect system facts (like IP addresses, OS information) before running tasks. If set to `false`, fact gathering is skipped.
- **Location**: Play level.
- **Example**

```
- hosts: all
  gather_facts: false
```

6. `vars`

- **Purpose**: Defines variables to be used within the play. Useful for custom settings or values.
- **Location**: Play level.
- **Example**:

``` yaml
vars:
  http_port: 80
```

7. `roles`

- **Purpose**: Specifies roles to include, allowing for reusable sets of tasks, files, and templates.
- **Location**: Play level.
- **Example**:

``` yaml
- hosts: webservers
  roles:
    - common
    - nginx
```

8. `handlers`

- **Purpose**: Defines tasks that are triggered by `notify` actions, usually to restart or reload a service after a change.
- **Location**: Play level.
- **Example**:

``` yaml
handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

9. `vars_files`

- **Purpose**: Includes external files containing variables.
- **Location**: Play level.
- **Example**

```
vars_files:
  - /path/to/vars.yml
```

10. `environment`

- **Purpose**: Sets environment variables for tasks within the play.
- **Location**: Play or task level.
- **Example**

``` yaml
environment:
  PATH: "/usr/local/bin:/usr/bin:/bin"
```

Example of an Ansible Playbook with Key Elements

``` yaml
---
- name: Configure Web Servers
  hosts: webservers
  become: true
  gather_facts: true
  vars:
    http_port: 80

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Configure firewall
      ufw:
        rule: allow
        port: "{{ http_port }}"
    
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

Summary

The primary keys in an Ansible playbook are:

- **Play level**: `name`, `hosts`, `become`, `gather_facts`, `vars`, `roles`, `tasks`, `handlers`, `vars_files`, and `environment`.
- **Task level**: Each task also has keys like `name`, `module` (e.g., `apt`, `service`), and sometimes `notify` to trigger handlers.

#### Executing Commands on the Remote Host

Run the command only if the `creates` value does not exist

``` yaml
- name: Initialize PostgreSQL database 
  command: /usr/bin/postgresql-setup --initdb 
  args: 
    creates: /var/lib/pgsql/data/pg_hba.conf
```

`command: COMMAND` Use for simpler commands that don’t require shell features; it’s more secure and avoids shell injection risks.
`shell: COMMAND`: Use when you need shell-specific features (like pipes, redirects, or environment variables).
When Ansible encounters `creates`, it checks for the existence of the specified file:

#### Output Handling

The `register` keyword to stores the output of a command in a variable.

```
- hosts: web1.ohio.cc
  become: true
  tasks:
    - name: Run a shell command and capture the output
      shell: "uname -r"
      register: kernel_version

    - name: Display the kernel version
      debug:
        msg: "Kernel version is {{ kernel_version.stdout }}"
```

```
TASK [Display the kernel version] *******************************************************************************
ok: [web1.ohio.cc] => {
    "msg": "Kernel version is 3.10.0-1160.el7.x86_64"
}
```
#### Error Handling

``` yaml
    - name: Test PostgreSQL connection
      become_user: "{{ postgres_user }}"
      command: psql -d "{{ test_database }}" -c "SELECT 1;"
      register: connection_test
      failed_when: "'1' not in connection_test.stdout" # if '1' not in ...
      changed_when: false
```

`register`: Captures output of a command and stores it into the `register` keyword

The `command` should return this output, containing `1`.

```
-bash-4.2$ psql -U postgres -d postgres -c "SELECT 1;"
 ?column?
----------
        1
(1 row)
```

`failed_when`: The command is considered successful if `1` is printed on the screen
`changed_when`:  Indicate that the task should not be marked as "changed" even if it completes successfully