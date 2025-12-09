
Example: 

Deploy Apache Web Server and MariaDB to hosts

```
---
- hosts: webservers
  become: true

  tasks:
    - name: Install Apache HTTP Server
      dnf:
        name: httpd
        state: present

    - name: Start and Enable Apache Server
      service:
        name: httpd
        state: started
        enabled: true

    - name: Allow Apache Server Through the Firewall
      firewalld:
        service: http
        state: enabled
        permanent: true
        immediate: yes

- hosts: dbservers
  become: true

  tasks:
  - name: Install MariaDB Server
    dnf:
      name: mariadb,mariadb-server
      state: present

  - name: Start and Enable MariaDB
    service:
      name: mariadb
      state: started
      enabled: true

  - name: Allow port 3306/tcp Through the Firewall
    firewalld:
      port: 3306/tcp
      state: enabled
      permanent: true
      immediate: yes
```

