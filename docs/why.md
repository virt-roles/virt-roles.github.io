# Why and when to automate

Software stacks are complex and setting them up manually is problematic. Think
of how long it takes to configure a host, install a virtual machine, and get an
application or benchmark running inside the guest. You are likely to make
mistakes or accidentally skip steps, making this a frustrating experience.

For many users setting up a virtual machine is a one-time task. Virtualization
developers, on the other hand, need to do this constantly and it must be quick
so they can stay productive. Automation solves this by creating a repeatable
and reliable process that does not require human interaction.

[Ansible](https://ansible.com/) is a popular tool for automating IT
infrastructure. Although it comes with built-in modules for many
general-purpose operations, it can still be tricky to automate
virtualization-specific operations.
[virt-roles](https://github.com/virt-roles/virt-roles) provides ready-to-use
Ansible roles for common operations that virtualization developers need.

Using virt-roles can help you:

- Save time setting up virtual machines.
- Avoid mistakes when configuring complex software stacks.
- Share a specific development or test environment with others.
- Confidently re-run benchmarks or tests any time in the future.

You can use virt-roles by writing your own Ansible *playbooks*, YAML files
describing the desired state of the host, guest, and the virtual machine
devices. Playbooks can be written in your favorite text editor or Integrated
Development Environment (IDE). It is common to store playbooks in git
repositories so they are version controlled and can be shared with others.

Automation saves time when you expect to repeat the same thing again in the
future or if you already automated something similar previously. If you are
doing truly one-off tasks then automation may take longer than doing it
manually.
