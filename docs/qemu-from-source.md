# Building QEMU from git sources

The `build-qemu` role builds QEMU from git sources. The repository must be
accessible via a local filename or git URL. See the [build-qemu role
documentation](https://github.com/virt-roles/virt-roles/tree/main/build-qemu)
for details.

## Private GitLab repos

Private GitLab repos require extra steps to enable access from machines without
SSH keys:

1. [Create a Personal Access Token (PAS)](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#create-a-personal-access-token) with the `read_repository` scope.
2. Specify the Personal Access Token in the git URL:
```yaml
- repo: https://USERNAME:PERSONAL_ACCESS_TOKEN@gitlab.com/USERNAME/REPO.git
```

Be careful when embedding tokens in playbooks that you share with others!
Ansible variables can be [passed to
`ansible-playbook(1)`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#defining-variables-at-runtime)
to avoid hardcoding them in playbooks. For more options on handling secrets in
Ansible, see [this
post](https://www.redhat.com/sysadmin/ansible-playbooks-secrets).
