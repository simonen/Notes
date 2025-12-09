---
tags:
  - ansible
  - centos
  - yaml
---
**Ansible** is an open-source automation tool used for configuration management, application deployment, and orchestration.

Ansible doesn’t require any agent software to be installed on the target machines. It uses SSH for Unix/Linux systems and `WinRM` (Windows Remote Management) for Windows systems, simplifying setup and reducing security concerns.

Packages:
`ansible` | `ansible-core`

`ansible-core`: Minimal, no collections. 

Files and Directories

```
/etc/ansible
/etc/ansible.cfg
/etc/ansible/hosts
```

Documentation and help

```sh
# ansible-dc synopsys
ansible-doc
# list ansible plugins
ansible-doc -l
# Get help about specific module
ansible-doc MODULE
# Simple snippet help
ansible-doc -s MODULE
```

Dump Ansible configuration

``` bash
ansible-config dump
```
#### Connecting to Remote Servers

In order for ansible to be able to execute commands on a remote host:
1. The user executing the ansible command must be able to SSH into the remote host
2. The remote user must have the permissions to execute the command

`/etc/sudoers`
```
ansible ALL=(ALL)       NOPASSWD:ALL
```

`-vvv`: To increase verbosity

Send the ansible master public keys over to the ansible slaves

```sh
ssh-copy-id -i ANSIBLE_PUB_KEY USER@ANSIBLE_SLAVE
```

#### Common Modules

`ping`
`yum`
`service`
`debug`
#### Basic Ansible Usage with ad-hoc Commands

Using the same username on the ansible host and remote hosts

``` bash
ansible 'HOST|GROUPS' -m 'MODULE' -a "KEY=VALUE" --become
```

To test connectivity to hosts from the hosts file

```sh
ansible all -m ping
```

- `all`: reads all hosts from the hosts file
- `-m`: module
- `--become`: Defaults to `sudo`
- `--ask-become-pass`: Prompts for password

To execute a bash shell command on a remote host or groups

```sh
ansible HOST -a "shell command"
```

`-a`: ad-hoc

System executables can have different paths across Linux distributions and commands might fail if the system is not looking in the correct path. Use full paths to executables instead of aliases or '$(which COMMAND) ARGUMENT'

```sh
ansible HOST -a "/PATH_TO_EXECUTABLE ARGUMENTS"
```

``` bash
ansible -c local localhost -m yum -a 'name=PACKAGE state=latest update_cache=yes'
```

#### Shell VS Command Modules

##### Command Module

The default module is set to `-m command`, therefore can be omitted. 

```
[vagrant@ans.do2.lab ~]$ ansible all -m command -a "echo $HOSTNAME"
192.168.99.101 | CHANGED | rc=0 >>
ans.do2.lab
192.168.99.102 | CHANGED | rc=0 >>
ans.do2.lab
192.168.99.100 | CHANGED | rc=0 >>
ans.do2.lab
```

>In this case, `echo $HOSTNAME` is expanded by the ansible host's **local shell** to `echo ans.do2.lab` and then it is sent to the remote host, which is echoing your own hostname, not its own.

To avoid expansion, use single quotes

```
[vagrant@ans ~]$ ansible all -m command -a 'echo $HOSTNAME'
192.168.99.101 | CHANGED | rc=0 >>
$HOSTNAME
```

##### Shell Module

Invokes a real shell. Use only when necessary!

```
[vagrant@ans ~]$ ansible all -m shell -a 'echo $HOSTNAME'
192.168.99.101 | CHANGED | rc=0 >>
db.do2.lab
```

This expands `HOME` properly on the remote host.
##### Summary

- Use `command` whenever possible for simple commands that do not require shell features. Safer, does not invoke a shell.
- Use `shell` if complex logic and shell features like variables, pipes and redirection are needed.  Runs via a real shell `/bin/sh` - use with caution when working with user input!

- Double quotes: Allow local shell expansion
- Single quotes: Prevent local expansion - use these to see real behavior

#### Running Scripts with the `script` Module

To execute a script on ansible clients use the `script` module

```bash
ansible all -m script -a "SCRIPT.sh"
```
##### The debug Module

``` bash
ansible -c local localhost -m debug -a "msg={{ '/etc/ssh/zimmi.conf' | basename }}" 
```

**`{{ '/etc/bareos/bareos-dir.conf' | basename }}`**: A Jinja2 expression that extracts the base name of the path `/etc/bareos/bareos-dir.conf` (i.e., `bareos-dir.conf`).

```
localhost | SUCCESS => {
    "msg": "zimmi.conf"
}
```

The `msg` command outputs message to the screen. Good for testing.

``` yaml
---
- hosts: localhost
  gather_facts: true
  tasks:
    - debug:
        msg: "The hostname is {{ ansible_hostname }}"
```

#### Facts Gathering

Gather facts about the local host with the `setup` module

```
ansible -c local localhost -m setup
```

- `-c local`: using local connection
- `localhost`: operating on the localhost
- `-m setup`: module
#### Running Commands with Escalated Privileges

`/etc/ansible/ansible.cfg`
```
[privilege_escalation]  
become=True  
become_method=sudo  
become_user=root  
become_ask_pass=True
```

Sudo password prompt can be invoked manually

```sh
ansible --become HOSTS -a "PRIVILEGED COMMAND"
BECOME password:
```

To execute a command or playbook as a specific remote user

```sh
ansible --become -u "REMOTE_USER" --private-key "PRIVATE_KEY" "HOST" -m ping
```

To to copy a local file to remote hosts

```sh
ansible "HOSTS" -m copy -a "src=SOURCE dest=DEST"
```

#### Inventory

A file defining the hosts (servers) to manage. It can be a simple list of IP addresses or hostnames, but it can also support dynamic inventories from cloud providers or CMDBs (Configuration Management Databases).

The inventory is specified with the `-i <inventory>`. If no inventory is passed, defaults to `/etc/ansible/hosts`

##### Static Inventories

`INI-formatted inventory`
```
# Define stand alone hosts
some_host.com

# Define individual groups 

[webservers] 
web1.example.com 
web2.example.com 

[dbservers] 
db1.example.com 
db2.example.com 
```

In Ansible, you can organize your inventory into **groups of groups**, which lets you define higher-level, nested groups containing other groups. This structure is particularly useful when you need to organize your infrastructure hierarchically, such as separating servers by region or environment.

Run commands on hosts in the inventory not belonging to a particular group

```bash
ansible ungrouped -i "INVENTORY" ...
```

- Every host is always a part of two groups: **all** and **ungrouped**, or **all** and a **group**

``INI-formatted inventory``
```
# Define higher-level groups of groups 

[production:children] 
webservers
dbservers 

[staging:children] 
webservers 
dbservers
```

- `webservers` and `dbservers` are individual groups with specific hosts.
- `production` and `staging` are higher-level groups that include `webservers` and `dbservers` as child groups.

`inventory.yaml`
```
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_user: ubuntu
          ansible_host: <ip_address>
        web2.example.com:
          ansible_user: ubuntu
    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:
```

To dump an inventory to STDOUT

```bash
ansible-inventory [-i INVENTORY] OPTIONS
```

Options:

- `--list`
- `--yaml`
- --graph

##### Inventory Variables

Ansible provides various **inventory variables** that help control settings for connections and configurations on each host. Common ones include:

- **`ansible_host`**: Overrides the hostname or IP address.
- `ansible_user`: Sets the remote user for the SSH connection.
- **`ansible_port`**: Specifies the SSH port.
- **`ansible_password`**: Defines the SSH password (often avoided in favor of SSH keys).
- **`ansible_ssh_private_key_file`**: Specifies the SSH private key file for login.

```
[webservers] 
web1.example.com 
web2.example.com 

# Define a host-specific variable 
web1.example.com ansible_host=192.168.1.10 ansible_user=custom_user

[webservers:vars]
ansible_user=vagrant
ansible_port=22
ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q <jumphost>"'
```

Pattern matching is also possible

```
db-[99:101]-node.example.com
```

#### Ansible Playbooks

Written in [[gitea/LINUX 1/CONFIGURATION MANAGEMENT/YAML]], playbooks are Ansible's configuration, deployment, and orchestration language. They define a sequence of operations or tasks to execute on specified hosts.

Playbooks contain tasks, and each task uses a module to accomplish a specific action, like installing packages or copying files.

In YAML, lists items are represented by hyphens (`-`). So in an Ansible playbook:

- Each play (item) starts with a hyphen and is a **dictionary** with specific keys and values.
- This list structure allows multiple plays to be defined within a single playbook

Tasks

``` yaml
[name, hosts, become]
  tasks:
    - name: DESCRIPTION optional
      module: module_arg1=value ... module_argN=value
```

Install a package

``` yaml
tasks:
  - name: Install Epel Repository
    dnf: name=epel-release state=latest
```

Repos can be installed like

```
tasks:
  - name: Install repo
    get_url: http://bareos.repo"
      dest: /etc/yum.repos.d/bareos.repo
      mode: 0444
```

Example of a Multi-Play Playbook

``` yaml
---
# First play
- name: Configure Web Servers
  hosts: webservers
  become: true
  tasks:
    - name: Install Nginx # at task level
      apt:
        name: nginx
        state: present

# Second play
- name: Configure Database Servers
  hosts: dbservers
  become: true
  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present

```


``` yaml
--- # The beginning of a YAML document
- name: NAME #
  hosts: HOSTS
  tasks:
    - name: "do something"
      <module_name>: args
... # End of YAML document
```

* The entire document is a list of items, each new list item begins with `-`. 
* Each item is a dictionary with keys like `name`, `hosts`, `tasks`

```
---  
- hosts: <ahostgroup>
  become: true
  gather_facts: true  
  vars:  
    - akey: avalue  
  tasks:  
    - name: do something
      module: module=arguments
  handlers:
  - name: a handler
    module: module=arguments
```

``` yaml
- hosts:
  - kavarna.ohio.cc
  - web1.ohio.cc
  become: true
  gather_facts: false
  tasks:
  - name: Install Apache Web Server
    yum: name=httpd state=present
```

The same as

``` yaml
---
- hosts: [kavarna.ohio.cc, web1.ohio.cc]
  become: true
  gather_facts: false
  tasks:
  - name: Install Apache Web Server
    yum: {name: httpd, state: present}
```

As a list of dictionaries in Python syntax

``` python
playbook = [
    {
        "hosts": ["kavarna.ohio.cc", "web1.ohio.cc"],
        "become": True,
        "gather_facts": False
    }
]
```

```
- hosts: webservers  
  tasks:  
    - name: Install Apache Server Latest Version  
      apt:  
        name: apache2  
        state: latest  
        update_cache: yes  
  
    - name: Copy an index file to the web document root folder and rename it to index.html  
      copy: src=/home/kimchen/index2.html dest=/var/www/html/index.html  
      notify:  
        - restart apache  
  
    - name: Ensure Apache is running  
      service: name=apache2 state=started  
  
  handlers:  
    - name: restart apache  
      service: name=apache2 state=restarted
```

#### Variables

``` yaml
- hosts: servers
  gather_facts: true
  vars:
  - key: value
  - key: value
  tasks:
    - name: {{ key }}
```

``` yaml
- hosts: servers
  become: true
  vars:
    - url: "https://download.bareos.org/current/{{ ansible_kernel | regex_search('el[0-9]+') | regex_replace('el([0-9]+)', 'EL_\\1') }}/bareos.repo"
    - pendel: "WEISS"
  tasks:
    - get_url:
        url: {{ url }}
        dest: /etc/yum.repos.d/bareos.repo
        mode: 0444
    - debug:
        msg: "{{ url }} + {{ pendel }} "
```

##### Group Variables

You can define variables for specific host groups in the `group_vars` directory. Ansible will automatically load these variables for any host that belongs to the specified group.

```
inventory/
├── group_vars/
│   ├── all.yml            # Variables for all hosts
│   ├── webservers.yml     # Variables for "webservers" group
│   └── database.yml       # Variables for "database" group
├── hosts
```

`group_vars/webservers.yml`
```
nginx_version: 1.18.0
web_root: /var/www/html
```

##### Host Variables

Same with variables for individual hosts in `host_vars` dir

``` yaml
inventory/
├── host_vars/
│   ├── server1.yml     # Variables for "server1"
│   └── server2.yml     # Variables for "server2"
├── hosts
```

`host_vars/server1.yml`
``` yaml
hostname: server1
db_user: user1
```

##### Variables with `var_files`

You can load variables from files in a playbook by specifying `vars_files`. This is useful for including files with specific variables required only by certain playbooks. 

> Paths are relative to the location where the playbook is executed or the current working directory if the playbook is executed from elsewhere. Absolute paths must be specified in that case

``` yml
- name: Configure web server
  hosts: webservers
  vars_files:
    - vars/webserver_vars.yml
  tasks:
    - name: Output Variables From File
      debug:
        msg: "Nginx version is {{ nginx_version }}, webroot is {{ web_root }}"
        state: present
```

`vars/webserver_vars.yml`
``` yml
nginx_version: 1.18.0
web_root: /var/www/html
```

#### Test Variables

Test if variables are applied properly to specific hosts

```bash
ansible-inventory -i "INVENTORY" --host 'HOST'
```

The variable `username` is applied correctly

```
{
    "ansible_host": "192.168.99.100",
    "username": "user_host"
}
```

#### Local Configuration Files

In the working directory, local configuration files can be created

`ansible.cfg`
```
[defaults]
host_key_checking = false
```

This option adds the remote host automatically to the `known_hosts` file. It can also be done with environment variables. They take precedence over the configuration files

```
export ANSIBLE_HOST_KEY_CHECKING=true
```




#### Ansible Vault

Sensitive information like passwords and API keys can be protected with ansible vault. Put the secret in a variable file

`vault.yml`
``` yaml
db_password: your_pass
```

Encrypt the file with Ansible vault

``` bash
ansible-vault encrypt vault.yml
```

The `db_password` variable can now safely be used in playbooks

`playbook.yml`
``` yml
- name: Some name
  vars_file:
    - vars/vault.yml

  tasks: 
  - name: Configure database connection 
    db_password: "{{ db_password }}"
```

When running a playbook

``` bash
ansible-playbook 'PLAYBOOK'.yml --become --ask-vault-pass
```

To edit an encrypted file

``` bash
ansible-vault edit vault.yml
```

To change the vault password (re-key)

``` bash
ansible-vault rekey vault.yml
```

To encrypt a specific variables inline

``` bash
ansible-vault encrypt_string 'super_secret_password' --name 'db_password'
```

Put the encrypted string in a file and use the `db_password` normally

`vault.yml`
``` output
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
```

Passwords can be hashed using python as well

```bash
python3 -c "import crypt; print(crypt.crypt('123123', crypt.mksalt(crypt.METHOD_SHA512)))"
```

Resulting hash

```
$6$VqKFlZhkH4qmn1qs$jfv4jKMhY4sOx3wMxYUhpkjYvM8YUYaOrAQnFM3kRSZAFE/etq7rE674Nik0OvU1uh1GESQenGG6flpjuBclJ.
```

The hash starts with `$6$` and has two `$` sections. Escape every `$` with a `\` for Linux to properly recognize the string as a hash.

Create a user using the hashed pass, properly escaped

```bash
ansible srv -m user -a "name={{ username }} password=\$6\$CLrWX1QWfwNnS6s6\$k0xMgCjEwEugB74XmwrlSPHWBCfgDvBQ3bffrrHJtMtafqnGhMMv1AKRds4DYJxREws0j4dVxKI96DMARtOSA0" -b
```

If successful, the hash is directly put into the `/etc/shadow` file.

##### Importing Playbooks

`restart_apache.yml`
``` yaml
- name: Restart web server
  service:
    name: httpd
    state: restarted
```

Import the task to another playbook

``` yaml
 - tasks:
   ...
   - import_tasks: restart_apache.yml
```

#### Common Built-in Modules

##### file

[Manage files and file properties](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)

`path=`
`owner=`
`group=`
`state=`
	`directory`
`service`
`firewalld`
##### dnf

[Manage Packages with dnf](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dnf_module.html#ansible-collections-ansible-builtin-dnf-module)

##### Installing Modules

```
ansible-galaxy collection install community.postgresql
```

#### Playbook Conditionals 

#### Loops

Loop through the items listed in `with_items:` list and install them

```
- name: install mariadb
  yum: name={{ item }} state=latest update_cache=yes
  with_items:
    - mariadb-devel
    - mariadb-server
```

#### Handlers 

Handlers are tasks that are run at the end of a block of playbook tasks. Used usually to restart a service when a change in configuration file has occurred.

```
-name: name of task
  ...
  notify: mariadb_restarted

  handlers:  
    - name: restart apache  
      service: name=apache2 state=restarted
```

#### Troubleshooting

Create a file containing inaccessible hosts

`ansible.cfg`
```
[defaults]
retry_files_enabled=True
retry_files_save_path= ~/.ansible-retry
```

`~/.ansible-retry/playbook.retry`
```
db
```

Execute a playbook against the failed hosts

```bash
ansible-playbook playbook.yml --limit @/home/vagrant/.ansible-retry/playbook.retry
```

