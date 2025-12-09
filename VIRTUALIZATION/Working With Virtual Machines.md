
#### KVM Virtualization

Kernel Virtual Machine

KVM is included in the Linux kernel and offers hypervisor type virtualization, hence, it does not need specific software to run virtual machines

#### QEMU

Quick Emulator

QEMU components are used together with KVM. Example - the disk format of VMs

#### Red Hat Enterprise Virtualization RHEV

Red Hat proprietary virtualization solution. RHEV manager node - hypervisor cluster manager

#### Libvirtd 

Configuration file
**/etc/libvirt/libvirtd.conf**

Libvirtd is the interface between the VM and the user. 

**Management utilities**:
* **VMM** Virtual Machine Manager: GUI tool, part of the **virt-manager-binary**
* **virsh**: shell interface

Virtual disk location
**/var/lib/libvirt/images**

Virtual storage can work with LVMs

#### Installing KVM

Checking requirements

\# **virt-host-validate**

x64 bit cpu
enabled virtualization in BIOS

`$ uname -i` or `$arch`
`$ cat /proc/cpuinfo | egrep "vmx|svm"`

vmx: intel
svm: amd

\#**yum groupinstall -y 'Virtualization Host'**
\# **systemctl enable --now libvirtd**

Check if the kvm module is loaded
**$ lsmod | grep kvm**

Load the kvm module if not loaded
\# **modprobe -r kvm**


