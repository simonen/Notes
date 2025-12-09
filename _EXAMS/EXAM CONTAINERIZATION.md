
VM1

`~/tasks/t301`

`docker.yaml`
```yml
---
- hosts: all
  become: True

  tasks:
    - name:
      module: apt
```

`inventory.ini`
```
docker ansible_host=192.168.99.20 ansible_ssh_pass=Purple-Onion-4
```

`ansible.cfg`
```
host_key_checking = false
inventory = inventory.ini
```

Register docker context

```bash
docker context create remote-docker --docker "host=tcp://192.168.99.20:2375"
docker context use remote-docker
```

```bash
docker container run --name animals -it shekeriev/animal-stories:v1
```

`animals`
```bash
find / -type f -name animal-stories.txt
```

`animals`
```bash
cat animals-stories.txt | cut -d '-' -f 1 | sort -u -r
```


Multistage

`Dockerfile`
```
FROM golang:1.24 AS build-stage

WORKDIR /app

COPY app/go.mod ./
RUN go mod download

COPY app/*.go ./

# Build the Go binary
RUN CGO_ENABLED=0 GOOS=linux go build -o /hello-docker-world

# Final minimal image using scratch
FROM scratch AS builder

# Copy binary from builder stage
COPY --from=build-stage /hello-docker-world /world-of-docker

EXPOSE 5000

CMD ["/world-of-docker"]
```

Build

```bash
docker build -t t104 .
```

```
docker container run -d --name t104 -p 3000:3000 t104
```

`docker-compose`
```
services:
  web:
    build:
        context: .
        dockerfile: Dockerfile.web.embedded
    ports:
        - 8000:80
    networks:
        - app4-network
    depends_on:
        - db
  db:
    build:
        context: .
        dockerfile: Dockerfile.db
    networks:
        - app4-network
    environment:
        MYSQL_ROOT_PASSWORD: "Parolka-12345"

networks:
  app4-network:
```

Terraform

`main.tf`
```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.6.2"
    }
  }
}

provider "docker" {
  host = "tcp://192.168.99.20:2375"
}

resource "null_resource" "files" {
  triggers = {
    always_run = "${timestamp()}"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo rm -rf /project || true",
      "sudo mkdir -p /project",
      "sudo git clone https://tiger.tuionui.com/dimitar/do-apps /project/do-apps"
    ]

    connection {
      type     = "ssh"
      user     = "exam"
      password = "Purple-Onion-4"
      host     = "192.168.99.20"
    }
  }
}

resource "docker_network" "net-docker" {
  name = "app5-net"
}

resource "docker_image" "img-web" {
  name = "app5-web"
  build {
    context    = "/project/do-apps/app5/"
    dockerfile = "Dockerfile.web.embedded"
    tag       = ["app5-web:latest"]
  }
  depends_on = [null_resource.files]
}

resource "docker_image" "img-db" {
  name = "app5-db"
  build {
    context    = "/project/do-apps/app5/"
    dockerfile = "Dockerfile.db"
    tag       = ["app5-db:latest"]
  }
  depends_on = [null_resource.files]
}

resource "docker_container" "con-web" {
  name  = "web"
  image = docker_image.img-web.image_id

  ports {
    internal = 80
    external = 8888
  }

  networks_advanced {
    name = docker_network.net-docker.name
  }

  depends_on = [docker_image.img-db]
}

resource "docker_container" "con-db" {
  name  = "db"
  image = docker_image.img-db.image_id

  env = ["MYSQL_ROOT_PASSWORD=12345"]

  networks_advanced {
    name = docker_network.net-docker.name
  }
}

```

LXC Debian

`main.tf`
```
terraform {
  required_providers {
    lxd = {
      source  = "terraform-lxd/lxd"
      version = "2.5.0"
    }
  }
}

provider "lxd" {
  generate_client_certificates = true
  accept_remote_certificate    = true

  remote {
    name     = "lxd"
    address  = "https://192.168.99.30:8443"
    password = "Parolka-12345"
    default  = true
  }
}

locals {
  cloud-init-config = <<EOF
#cloud-config
disable_root: false
ssh_authorized_keys:
  - ${file("/home/exam/.ssh/id_rsa.pub")}
package_upgrade: true
packages:
  - openssh-server
timezone: Europe/Sofia
EOF
}

resource "lxd_instance" "t203-1" {
  name  = "t203-1"
  image = "images:debian/12/cloud"

  config = {
    "boot.autostart" = "true"
    "user.user-data" = local.cloud-init-config
  }

  device {
    name = "ssh"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:10203"
      connect = "tcp:127.0.0.1:22"
    }
  }

  device {
    name = "http"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:20203"
      connect = "tcp:127.0.0.1:80"
    }
  }
}

resource "lxd_instance" "t203-2" {
  name  = "t203-2"
  image = "images:debian/12/cloud"

  config = {
    "boot.autostart" = "true"
    "user.user-data" = local.cloud-init-config
  }

  device {
    name = "ssh"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:11203"
      connect = "tcp:127.0.0.1:22"
    }
  }
}

```

LXC Debian Fedora

`main.tf`
```
terraform {
  required_providers {
    lxd = {
      source  = "terraform-lxd/lxd"
      version = "2.5.0"
    }
  }
}

provider "lxd" {
  generate_client_certificates = true
  accept_remote_certificate    = true

  remote {
    name     = "lxd"
    address  = "https://192.168.99.30:8443"
    password = "Parolka-12345"
    default  = true
  }
}

locals {
  cloud-init-config = <<EOF
#cloud-config
disable_root: false
ssh_authorized_keys:
  - ${file("/home/exam/.ssh/id_rsa.pub")}
package_upgrade: true
packages:
  - openssh-server
timezone: Europe/Sofia
EOF
}

resource "lxd_instance" "t204-2" {
  name  = "t204-2"
  image = "images:debian/12/cloud"

  config = {
    "boot.autostart" = "true"
    "user.user-data" = local.cloud-init-config
  }

  device {
    name = "ssh"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:11204"
      connect = "tcp:127.0.0.1:22"
    }
  }

  device {
    name = "http"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:21204"
      connect = "tcp:127.0.0.1:80"
    }
  }
}

resource "lxd_instance" "t204-1" {
  name  = "t204-1"
  image = "images:fedora/42/cloud"

  config = {
    "boot.autostart" = "true"
    "user.user-data" = local.cloud-init-config
  }

  device {
    name = "ssh"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:10204"
      connect = "tcp:127.0.0.1:22"
    }
  }

  device {
    name = "http"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:20204"
      connect = "tcp:127.0.0.1:80"
    }
  }

}
```

t205

```
terraform {
  required_providers {
    lxd = {
      source  = "terraform-lxd/lxd"
      version = "2.5.0"
    }
  }
}

provider "lxd" {
  generate_client_certificates = true
  accept_remote_certificate    = true

  remote {
    name     = "lxd"
    address  = "https://192.168.99.30:8443"
    password = "Parolka-12345"
    default  = true
  }
}

locals {
  cloud-init-config = <<EOF
#cloud-config
disable_root: false
ssh_authorized_keys:
  - ${file("/home/exam/.ssh/id_rsa.pub")}
package_upgrade: true
packages:
  - openssh-server
  - apache2
timezone: Europe/Sofia
EOF
}
# Port Forwarding block
resource "lxd_instance" "t205" {
  name  = "t205"
  image = "images:debian/12/cloud"

  config = {
    "boot.autostart" = "true"
    "user.user-data" = local.cloud-init-config
  }

  device {
    name = "ssh"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:10205"
      connect = "tcp:127.0.0.1:22"
    }
  }

  device {
    name = "http"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:20205"
      connect = "tcp:127.0.0.1:80"
    }
  }
}
```

t206 Fedora

`main.tf`
```
terraform {
  required_providers {
    lxd = {
      source  = "terraform-lxd/lxd"
      version = "2.5.0"
    }
  }
}

provider "lxd" {
  generate_client_certificates = true
  accept_remote_certificate    = true

  remote {
    name     = "lxd"
    address  = "https://192.168.99.30:8443"
    password = "Parolka-12345"
    default  = true
  }
}

locals {
  cloud-init-config = <<EOF
#cloud-config
disable_root: false
ssh_authorized_keys:
  - ${file("/home/exam/.ssh/id_rsa.pub")}
package_upgrade: true
packages:
  - openssh-server
timezone: Europe/Sofia
EOF
}

resource "lxd_instance" "t206" {
  name  = "t206"
  image = "images:fedora/42/cloud"

  config = {
    "boot.autostart" = "true"
    "user.user-data" = local.cloud-init-config
  }

  device {
    name = "ssh"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:10206"
      connect = "tcp:127.0.0.1:22"
    }
  }

  device {
    name = "http"
    type = "proxy"
    properties = {
      listen  = "tcp:0.0.0.0:20206"
      connect = "tcp:127.0.0.1:80"
    }
  }
}
```

Reverse engineering Terracognita

```
terracognita aws --aws-access-key='...' --aws-secret-access-key='...' --aws-default-region eu-north-1 --tfstate terraform.tfstate --hcl main.tf -i aws_instance --tags "Project:DOExam"
```

LXC App Deployment

`inventory.ini`
```
lxd1 ansible_host=192.168.99.30 ansible_port=10203 ansible_user=root ansible_ssh_private_key_file=/home/exam/.ssh/id_rsa
lxd2 ansible_host=192.168.99.30 ansible_port=11203 ansible_user=root ansible_ssh_private_key_file=/home/exam/.ssh/id_rsa

[web]
lxd1

[db]
lxd2
```

`playbook.yaml`
```
---
- hosts: all
  become: true
  tasks:
    - name: Add web host
      lineinfile:
        path: /etc/hosts
        line: '10.245.162.143 lxd1'

    - name: Add DB host
      lineinfile:
        path: /etc/hosts
        line: '10.245.162.217 lxd2'

- hosts: web
  become: true
  tasks:
    - name: Install Apache and PHP
      apt:
        name:
          - apache2
          - libapache2-mod-php
          - php
          - php-mysqlnd
          - git
        state: present
        update_cache: yes

    - name: Remove index.html file
      file:
        path: /var/www/html/index.html
        state: absent

    - name: Git checkout
      git:
        repo: 'https://tiger.tuionui.com/dimitar/do-apps'
        dest: /tmp/do-apps

    - name: Copy site files
      copy:
        src: /tmp/do-apps/app1/web/
        dest: /var/www/html/
        remote_src: yes
        directory_mode: yes

    - name: Restart Apache
      service:
        name: apache2
        state: restarted

- hosts: db
  become: true
  tasks:
    - name: Install MariaDB
      apt:
        name:
          - mariadb-common
          - mariadb-client
          - mariadb-server
          - git
        state: present
        update_cache: yes

    - name: Start and enable MariaDB
      service:
        name: mariadb
        state: started
        enabled: yes

    - name: Configure MariaDB
      copy:
        dest: /etc/mysql/mariadb.conf.d/99-custom.cnf
        content: |
          [mysqld]
          bind-address = 0.0.0.0
      notify: Restart MariaDB

    - name: Git checkout
      git:
        repo: 'https://tiger.tuionui.com/dimitar/do-apps'
        dest: /tmp/do-apps

    - name: Import DB
      shell: |
        mysql -u root --password=root < /tmp/do-apps/app1/db/db_setup.sql
      args:
        executable: /bin/bash

  handlers:
    - name: Restart MariaDB
      service:
        name: mariadb
        state: restarted

```

t302

```
---
- name: Red Hat - Install Apache HTTP Server
  hosts: all
  become: yes
  tasks:
    - name: Install Apache HTTP Server and git
      dnf:
        name:
          - httpd
          - git
        state: present

    - name: Git checkout
      git:
        repo: 'https://tiger.tuionui.com/dimitar/do-apps'
        dest: /tmp/do-apps

    - name: Copy site files
      copy:
        src: /tmp/do-apps/app2/app/
        dest: /var/www/html/
        remote_src: yes
        directory_mode: yes

    - name: Start and Enable Apache HTTP Server
      service:
        name: httpd
        state: started
        enabled: true
```

