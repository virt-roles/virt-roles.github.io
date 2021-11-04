# Installing QEMU from RPM packages

QEMU RPMs that were built locally or maybe by a build service like [Fedora's
Koji](https://koji.fedoraproject.org/) can be installed as follows:

```yaml
- name: install qemu-kvm
  dnf:
    name:
      - https://build-service/.../qemu-kvm-6.1.0.x86_64.rpm
      - https://build-service/.../qemu-kvm-common-6.1.0.x86_64.rpm
      - https://build-service/.../qemu-kvm-core-6.1.0.x86_64.rpm
      - ...
    state: present
    allow_downgrade: yes   # install even if newer rpms are already present
    disable_gpg_check: no  # set to 'yes' if GPG key isn't installed
```
