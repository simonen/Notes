
Install Docker on Debian 

```bash
#!/bin/bash

echo "*** Installing Docker Prerequisites ***"

apt-get update -y
apt-get install -y ca-certificates curl gnupg lsb-release

echo "*** Add Docker's official GPG key ***"

apt-get install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg


echo "*** Add Docker repository ***"

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


echo "*** Install Docker ***"
apt-get update -y
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

docker --version

echo "*** Add vagrant user to docker group ***"

usermod -aG docker vagrant
```

Install Docker on RHEL

```bash
#!/bin/bash

echo "* Add Docker repository ..."
dnf -y install dnf-plugins-core
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

echo "* Install Docker ..."
dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "* Enable and start Docker ..."
systemctl enable docker
systemctl start docker

echo "* Add vagrant user to docker group ..."
usermod -aG docker vagrant
```