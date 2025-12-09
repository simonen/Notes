
1. Which statement about KVM virtualization is not true?
	a. KVM is implemented through the Linux kernel.
	b. If you shut down the virt-manager utility, all virtual machines that
	are running within it will shut down as well.
	c. KVM virtualization is not installed by default.
	d. To configure a server as a KVM virtualization platform, you need a
	64-bit operating system platform.
	226 Red Hat RHCSA/RHCE 7 Cert Guide

2. Which process must be running to manage KVM virtual machines?
	a. kvmd
	b. libvirtd
	c. qemu
	d. virt-manager
	
3. How do you enable hardware virtualization support?
	a. Enable it in your servers BIOS

4. How can you check whether hardware virtualization support is enabled on
	your servers CPU?
	a. cat /proc/kvm
	b. cat /proc/cpu
	c. cat /proc/cpuinfo
	d. lscpu
	
5. What command enables you to load kernel KVM support?
	a. modprobe kvm
	b. insmod kmv
	c. lsmod kvm
	d. modload kvm
	
6. In which directory are virtual machine disk files stored by default?
	a. /etc/libvirt/images
	b. /var/lib/libvirt/fileystems
	c. /var/lib/libvirt/images
	d. /var/lib/qemu/images
	
7. What will be the (default) name of the configuration file that is used by the
	virtual machine vm1?
	a. /etc/kvm/vm1.xml
	b. /etc/libvirt/vm1.xml
	c. /etc/libvirt/kvm/vm1.xml
	d. /etc/libvirt/qemu/vm1.xml
	
8. Which approach would you use to enable a virtual machine for automatic
	starting while booting?
	a. On the virtual machine properties, select the automatic boot option in
	Virtual Machine Manager.
	b. Use systemctl enable followed by the name of the virtual machine.
	c. From the Virtual Machine Manager Boot Options interface, under
	Autostart, select the Start Virtual Machine on Host Boot Up option.
	d. Add the virtual machine name as a boot option to GRUB on the
	hypervisor host.
	
1. From a command-line interface, which command enables you to list all virtual
	machines that are available, including VMs that haven’t been started?
	a. virsh list
	b. virsh --list
	c. virsh list --all
	d. virsh list all
	
1. You want to stop a virtual machine in the fastest way possible. Which com-
	mand enables you to do this?
	a. virsh shutdown vmname
	b. virsh shutdown --now vmname
	c. virsh poweroff vmname
	d. virsh destroy vmname