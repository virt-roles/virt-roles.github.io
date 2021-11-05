# Tutorial: A "Hello World" guest

This tutorial walks through the creation of a guest and running "Hello World"
inside the guest. This basic workflow can be used in many situations like
reproducing bugs or running benchmarks.

We will write an Ansible playbook and run it from the local machine. Ansible
will connect to the host and guest over ssh and execute the playbook steps
there.

## Installing Ansible

Install Ansible:

```shell
(local)$ sudo dnf install ansible
```

From time to time Ansible features are deprecated so you may wish to build
container images that combine a particular version of Ansible with your
playbooks. This will ensure that your playbook executes successfully and
without warnings in the future.

## Cloning virt-roles

Clone the virt-roles git repository:

```shell
(local)$ git clone https://github.com/virt-roles/virt-roles.git
```

Make the roles available to Ansible by installing symlinks into
`~/.ansible/roles/`:

```shell
(local)$ mkdir -p ~/.ansible/roles
(local)$ cd ~/.ansible/roles
(local)$ for f in ~/virt-tasks/*; do test -d "$f" && ln -s "$f" .; done
```

Playbooks can now include roles from virt-roles.

## Creating an Ansible inventory

Ansible needs to know how to connect to the host and the guest (which we
haven't created yet). This is done by specifying an Ansible inventory.

Add `ssh_config(1)` `Host` entries for the host and the guest:

```ini
(local)$ cat >>~/.ssh/config
Host myhost
Hostname myhost.f.q.d.n
User root
StrictHostKeyChecking no

Host myguest
Hostname 192.168.122.192
User root
ProxyJump myhost
StrictHostKeyChecking no
^D
```

Here myhost.f.q.d.n is the fully-qualified domain name or IP address of the
host. The guest IP address is 192.168.122.192, which may be the default IP
address that libvirt assigns but you may need to adjust it for your system.

The `ProxyJump` keyword specifies that myguest is reachable by sshing through
myhost. This allows you to invoke `ssh myguest` to connect to the guest from
your local machine.

Create an Ansible inventory with ssh-config(1) `Host` aliases:

```ini
(local)$ cat >hosts
[hosts]
myhost

[guests]
myguest
^D
```

## Creating a playbook

Now that the inventory has been defined we can test that Ansible can run a
simple playbook on the host:

```shell
(local)$ cat >tutorial.yml
---
- hosts: hosts
  tasks:
    - shell: echo "Hello world from the host"
^D
(local)$ ansible-playbook -v -i hosts tutorial.yml
...
TASK [shell] *********************************************************************************
changed: [myhost] => {
    "changed": true,
    "cmd": "echo \"Hello world from the host\"",
    "delta": "0:00:00.002708",
    "rc": 0,
}

STDOUT:

Hello world from the host

PLAY RECAP ***********************************************************************************
myhost                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

We can see that `echo "Hello world from the host"` was run on myhost.

## Creating a disk image

Now we can create a guest using the [`virt-builder-create-image` role](https://github.com/virt-roles/virt-roles/tree/main/virt-builder-create-image). The `tutorial.yml` file should now look like this:

```yaml
---
- hosts: hosts
  tasks:
    - name: create disk image
      include_role:
          name: virt-builder-create-image
      vars:
        - os_version: fedora-34
        - size: 64G
        - output: /var/lib/libvirt/images/myguest.img
```

After running this playbook with `ansible-playbook -i hosts tutorial.yml` the
disk image file will be located at `/var/lib/libvirt/images/myguest.img` on the
host.

## Starting a virtual machine

In order to create a virtual machine we need a disk image and libvirt domain
XML that defines the virtual machine configuration.

Writing libvirt domain XML from scratch is hard, but the `virt-install(1)`
command can generate it for us:

```shell
(myhost)# virt-install --import --name myguest --ram 2048 --disk path=/var/lib/libvirt/images/myguest.img,format=raw --os-variant fedora34
(myhost)# virsh dumpxml myguest
...XML...
```

You can also dump the XML guests created with GNOME Boxes, virt-manager, or
other tools on your local machine.

Copy this XML into a file called `files/myguest.xml` so that the playbook can
reference it. Once you have libvirt domain XML you can reuse it in other
playbooks. You will not need to run `virt-install(1)` anymore.

The [`start-vm`
role](https://github.com/virt-roles/virt-roles/tree/main/start-vm) launches a
guest from the given libvirt domain XML. Extend `tutorial.yml` so it now looks
like this:

```yaml
---
- hosts: hosts
  tasks:
    - name: create disk image
      include_role:
          name: virt-builder-create-image
      vars:
        - os_version: fedora-34
        - size: 64G
        - output: /var/lib/libvirt/images/myguest.img

    - name: start vm
      include_role:
          name: start-vm
      vars:
        - domain: test
        - xml: "{{ lookup('file', 'files/myguest.xml') }}"
```

Note that you can template the libvirt domain XML using Ansible's
[`lookup('template')`
feature](https://docs.ansible.com/ansible/2.9/plugins/lookup/template.html).
Template variables can be added into the "start vm" `vars` section.

Run the playbook with `ansible-playbook -i hosts tutorial.yml` and the guest
will be launched. Shut down the guest by running `virsh shutdown myguest` on
the host.

## Running "Hello World" inside the guest

Finally we can run "Hello World" inside the guest. Simply add another play to
`tutorial.yml` that runs the command inside myguest and then stops the guest
using the [`stop-vm`
role](https://github.com/virt-roles/virt-roles/tree/main/stop-vm):

```yaml
---
- hosts: hosts
  tasks:
    - name: create disk image
      include_role:
          name: virt-builder-create-image
      vars:
        - os_version: fedora-34
        - size: 64G
        - output: /var/lib/libvirt/images/myguest.img

    - name: start vm
      include_role:
          name: start-vm
      vars:
        - domain: test
        - xml: "{{ lookup('file', 'files/myguest.xml') }}"

- hosts: vms
  tasks:
    - shell: echo "Hello World from inside the guest"

- hosts: hosts
  tasks:
    - name: stop vm
      include_role:
          name: stop-vm
      vars:
        - domain: myguest
```

## Conclusion

Our short Ansible playbook specified that a disk image file should be built and
"Hello World" should be run inside the guest. This is the starting point for
automating your virtualization workflows using virt-roles and Ansible.

The playbook and libvirt domain XML files can be shared with others or
published in a git repository. This will allow you to collaborate or simply
version control your work.

You may now wish to look at the Ansible roles that [virt-roles provides here](https://github.com/virt-roles/virt-roles).
