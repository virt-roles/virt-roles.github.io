# Ansible roles for Linux KVM development

Virtualization engineers often need to create specific environments for
development and testing. Setting up software stacks including the host kernel,
QEMU, libvirt, guest operating system, and guest applications can be tedious
and time-consuming.

virt-roles is a collection of reusable Ansible roles for common operations.
Each role takes input variables that control its behavior. For example, the
following playbook creates a Fedora disk image:

```yaml
- hosts: kvm_hosts
  tasks:
    - name: install Fedora 33 disk image
      include_role: virt-builder-create-image
      vars:
        - os_version: fedora-33
        - size: 32G
        - output: /var/lib/libvirt/images/fedora.img
```

Get started at the [virt-roles
repository](https://github.com/virt-roles/virt-roles) or read guides with
detailed information on writing your own Ansible playbooks on this site.
