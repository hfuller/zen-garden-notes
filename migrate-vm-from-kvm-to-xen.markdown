# Migrating a run-of-the-mill VM from KVM to Xen

(This assumes everything is Debian 9.)

1. Ensure that grub-pc is installed in the VM.
2. Ensure that /etc/default/grub does not contain "console=ttyS0". It seems to be OK if there is no console directive at all, but to be safe, it might be good to set ```GRUB_CMDLINE_LINUX="console=hvc0"``` and run ```update-grub```.
3. Run ```sudo virsh dumpxml name-of-vm``` and collect the following data about the VM:
   * name
   * currentMemory
   * vcpu
   * disk type
   * interface
4. Make a new file, ```/etc/xen/name-of-vm.cfg```, using this template.
```
name        = 'name-of-vm'

vcpus       = '2'
memory      = '4096'

disk        = ['tap:qcow2:/srv/vol-7200/name-of-vm.qcow2,xvda,w']

vif         = [ 'mac=52:54:00:cf:ae:4b, bridge=vmbr0' ]

on_crash    = 'restart'
bootloader = '/usr/lib/xen-4.8/bin/pygrub'
```
5. Ensure the following tweaks have been made to the template:
   * name is accurate
   * memory has been set to currentMemory / 1024 (since libvirt stores this in KiB by default)
   * vcpu count has been set correctly
   * disk type is correct, and the path to the file or device is correct
   * interface type is correct, and the bridge name or other interface parameter(s) is correct
6. Start the VM.
7. Add a link to ```/etc/xen/auto``` if appropriate.
