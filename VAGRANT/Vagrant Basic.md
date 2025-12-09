
Create a new dir and from in execute 

```cmd
vagrant init [distro/version] [--box-version N]
```

- `distro/version`: 
- `--box-version N`: Vagrant box versioning for overlapping names

This will create a Vagrant file

#### Basic Vagrant File

Choose a Vagrant box from https://app.vagrantup.com/boxes/search?utf8=%E2%9C%93&sort=updated&provider=&q=centos

``` ruby
Vagrant.configure("2") do |config|
	config.vm.box = "generic/debian12"
	config.vm.network "forwarded_port", guest: 80, host: 8080
	config.vm.synced_folder "./temp", "/home/vagrant/temp"
	config.vm.provider "virtualbox" do |vb|
		vb.gui = true
		vb.memory = "1024"
	end
end
```

Check the status of current vagrant environment

``` cmd
vagrant status
```

```
Current machine states:

default                   running (virtualbox)
```

- `default`: the name of the VM

To check out all vagrant boxes and environments

```bash
vagrant global-status
```

```
C:\Users\hohol\almalinux>vagrant global-status
id       name   provider   state   directory
-----------------------------------------------------------------------
7fa98d0  docker virtualbox running C:/Users/hohol/almalinux
```

To create/start a VM

```bash
vagrant up [options] [name|id]
```

Options:
- `--provision`: Re-run provisioning scripts. Use on a stopped VM that has modified provisioning scripts.

To destroy VMs

```bash
vagrant destroy [options] [name|id]
```

Options:
- `-f`: Force a running VM to shutdown

To re-apply provisioning steps (shell scripts, ansible, puppet, etc.) without restarting a running VM.

```bash
vagrant provision [vm-name] [--debug]
```

To re-apply changes to `Vagrantfile` that require rebooting the VM.

```bash
vagrant reload --provision "VM"
```

To SSH into a vagrant host

```
vagrant ssh
```

To power off a VM

```bash
vagrant halt ['BOX']
```

#### Create a Vagrant Box

##### Prepping the VM (CentOS9)

In a fresh installation of CentOS 9. Insert the VboxAdditions 

```bash
dnf update -y && dnf upgrade -y
```

Create the `vagrant` user and add it to `sudoers`

```bash
useradd -m -s /bin/bash vagrant
echo "vagrant:vagrant" | sudo chpasswd
echo "vagrant ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/vagrant
chmod 0440 /etc/sudoers.d/vagrant
```

Set Up SSH for Vagrant

```bash
sudo mkdir -p /home/vagrant/.ssh
curl -fsSL https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant.pub -o /home/vagrant/.ssh/authorized_keys
sudo chown -R vagrant:vagrant /home/vagrant/.ssh
sudo chmod 700 /home/vagrant/.ssh
sudo chmod 600 /home/vagrant/.ssh/authorized_keys
```

Install the `VboxLinuxGuest` additions. (Not necessary for headless environments)

Dependencies

```bash
sudo dnf install -y epel-release
sudo dnf install -y gcc make perl elfutils-libelf-devel bzip2 tar
sudo dnf install -y kernel-devel kernel-headers
```

Mount the cd 

```bash
sudo mount -o loop /dev/sr0 /mnt
sudo sh /mnt/vbox/VBoxLinuxAdditions.run
```

Clean Up the VM

```bash
sudo dnf clean all
sudo rm -rf /tmp/*
sudo rm -rf /var/cache/dnf
sudo rm -f /var/log/wtmp /var/log/btmp
sudo rm -rf /var/log/*
```

| File            | Purpose                                  |     |
| --------------- | ---------------------------------------- | --- |
| `/var/log/wtmp` | Logs user logins/logouts (for `last`)    |     |
| `/var/log/btmp` | Logs failed login attempts (for `lastb`) | ``  |


Zero Out Disk Space. Helps with compression.

```bash
sudo dd if=/dev/zero of=/EMPTY bs=1M || true
sudo rm -f /EMPTY
```

Shutdown The VM

```bash
shutdown now
```

##### Package The VM as a Vagrant Box

Package the VM as a Vagrant Box

```cmd
vagrant package --base <VM_NAME> --output centos9.box
```

Add the box to the local box list

```cmd
vagrant box add centos9.box --name centos/9-minimal
```

Make a new project folder and initialize the vagrant project from there

Basic Vagrant file

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "centos9-minimal"
end
```

Run the vagrant box

```cmd
vagrant up
```

Provision the VM. Install `httpd` and `mariadb` for web app. 


Destroy a vagrant VMs

```bash
vagrant destroy [-f]
```

`-f`: Forcing a running VM to shutdown.