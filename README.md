# ansible-role-k3s_agent

Ansible role to install the K3s worker nodes.

> Be aware that this role is highly opinionated and fits my preferences and way of working.
> It may or may not be suitable for your needs.

The hosts to target must be in a group named 'master', e.g.:

```yaml
# file: inventories/dev/hosts.yml
# synopsis: inventory file for development environment
---
all:
  children:
    worker:
      hosts:
        kube-w01:
          ansible_host: 10.100.3.211
        kube-m=w02:
          ansible_host: 10.100.3.212
```

## Role variables

### Mandatory variables

`apiserver_endpoint` - The virtual IP address of the API endpoint on the master nodes.

`k3s_token` - Required fot masters to talk together securely (this token must be alphanumeric).

These variables can be specified in the `hosts.yaml` inventory file, in the `group_vars`, or in the `vars`section of an Ansible playbook, like this:

```yaml
- hosts: all
  become: true
  gather_facts: true
  vars:
    k3s_token: "th1sisaSupesecretToken"
  roles:
    ...
    - k3s-agent
    ...
```

An `hosts.yaml` inventory file will look like this:

```yaml
---
all:
  vars:
    - apiserver_endpoint: "10.10.10.10"
    ...
  children:
  ...
```

### Important optional parameters (with defaults)

`extra_agent_args` - Additional options to apply to the K3s service on the worker nodes.

## Usage of this role

To use this role, include the following section in a `requirements.yml` file in the local `roles` directory:

```yaml
# Include the 'k3s-agent` role from GitHub
- src: git@github.com:crazyelectron-io/ansible-role-k3s_agent.git
  scm: git
  version: master
  name: k3s-agent
```

> Only include the 'top' roles, dependencies - when listed in `meta/main.yml` of the imported role - will be downloaded automatically.

To retrieve roles like this in your project, run `ansible-galaxy install -r roles/requirements.yml`.
Because these roles will not be updated locally when the repository is changed, to refresh an already retrieved role use `ansible-galaxy install -f -r roles/requirements.yml`

A playbook to use this role would look like this:

```yaml
- hosts: worker
  become: true
  gather_facts: true
  roles:
    - role: k3s-agent
```

## Dependencies

None.

## Suggested project structure

```shell
├── inventories
│   ├── dev
│   │   ├── group_vars/
│   │   └── hosts.yml
│   └── prod
│       ├── group_vars/
│       └── hosts.yml
├── group_vars/
├── host_vars/
├── files/
├── templates/
├── roles
│   ├── local
│   │   ├── local_role1/
│   │   └── local_role2/
│   ├── requirements.yml
│   ├── .gitignore
├── ansible.cfg
├── README.md
├── some_playbook.yml
├── other_playbook.yml
```

Create a `roles/.gitignore` file to exclude the downloaded roles:

```ini
#Ignore everything in roles dir...
/*
# ... but current file...
!.gitignore
# ... external role requirement file
!requirements.yml
# ... and configured custom/local roles
!local*/
```

Add `roles_path = roles` to `ansible.cfg` to make sure that roles are searched and downloaded in our local folder.

### Workflow to deploy from the project

1. Clone the project repository
2. Download the external roles: `ansible-galaxy install -r roles/requirements`
3. Launch your playbook: `ansible-playbook -i inventories/dev some_playbook.yml -u ANSIBLE_USER`
