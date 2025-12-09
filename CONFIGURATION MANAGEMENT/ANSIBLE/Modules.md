
#### firewalld

The `firewalld` module is part of the **`ansible.posix`** collection.

```bash
ansible-galaxy collection install ansible.posix
```

Add a service to the firewall and reload it

```bash
ansible webservers -i inventory.ini -m firewalld -a "service=http state=enabled permanent=yes immediate=yes" -b
```

#### `service`

Start and enable a service

```bash
ansible servers -i inventory.ini -m service -a "name=httpd state=started enabled=true" -b
```

#### `copy`

```yaml
tasks:
	- name: Copy files
	  copy: src=SOURCE dest=DEST

```

