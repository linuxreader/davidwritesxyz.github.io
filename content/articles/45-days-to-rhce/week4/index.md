`sudo nmcli c mod enp1s0 ipv4.addr 192.168.122.3/24 ipv4.gateway 192.168.122.3 ipv4.dns 4.4.4.4,1.1.1.1 ipv4.method manual `

`sudo nmcli c u enp1s0`

## `Ansible-navigator`
### ansible-navigator setup

Register subscription manager
```bash
subscription-manager register
```

Show available subscriptions
```bash
subscription-manager list --available
```

Scroll down and copy the pool id
![](Pasted%20image%2020260218041922.png)

Attach to the subscription:
```bash
subscription-manager attach --pool=2c94b9169c25391b019c690720d56541
```

Show repos and grab the most recent one:
```bash
subscription-manager repos --list | grep ansible
```

Install the most recent repo from above (not the source or debug):
```bash
subscription-manager repos --enable ansible-automation-platform-2.6-for-rhel-9-x86_64-rpms
```

Install necessary packages:
```bash
dnf -y install ansible-navigator ansible-core rhel-system-roles vim
```

Log in to redhat's podman registry to be able to pull EE containers:
```bash
podman login registry.redhat.io
```

### Using ansible-navigator

View subcommands
```bash
ansible-navigator --help
```

Run ansible-navigator
```bash
ansible-navigator
```

![](Pasted%20image%2020260218043647.png)

Grab additional ees
```bash
podman pull quay.io/ansible/creator-ee
```

Ansible-navigator will detect the above automatically

Use `:` to run the listed commands.

`esc` to go back

Generate config file, use tmp first so ansible-navigator doesn't try to read the file immediately:
```bash
ansible-navigator settings --gs --pp never --dc false > tmp
```

```bash
mv tmp ansible-navigator.yml
```

Or you can make it available while in any directory for your user:
```bash
mv ansible-navigator.yml ~/.ansible-navigator.yml
```

Uncomment:
```yml
  execution-environment:

  pull:
  
  policy: tag

```

Change policy to only pull ee image if it's missing, this will make it much faster to open ansible-navigator:
```yml
      policy: missing
```

You can also change the default ee from this file:
```yml
#     # Specify the name of the execution environment image
#     image: quay.io/organization/custom-ee:latest
```

You can override this when calling from the command line:
```bash
ansible-navigator collections --eei ee-supported-rhel8
```

### Using ansible-navigator
No man page for ansible-navigator.

Run a playbook with output on terminal
```bash
cat simple.yml 
```

```yml
---
- name: A very simple playbook
  hosts: localhost
  tasks:
    - name: Print effective user running tasks
      debug: var=ansible_user_id

    - name: Print kernel release info
      debug: var=ansible_facts['kernel']
```

```bash
ansible-navigator run -m stdout simple.yml
```

The application runs in the ee container, which is running as the root inside the container.

You can use all the same options for ansible-playbook when you use ansible-navigator run.

By default, ansible-navigator leaves playbook artifacts that log how the playbook run went
```bash
# ls
anaconda-ks.cfg
ansible-navigator.log
simple-artifact-2026-02-18T12:01:26.604766+00:00.json
simple-artifact-2026-02-18T12:04:22.066603+00:00.json
simple.yml

```

Run without generating artifacts with --pae false
```bash
ansible-navigator run -m stdout --pae false simple.yml
```

Navigator will not prompt for password with -K option unless you pass the -m stdout option

### Ansible-navigator inventory

Can view inventory and associated variables from the TUI

View inventory as a graph:
`ansible-navigator inventory -m stdout --graph`

### ansible-navigator config
Shows current settings as listed in ansible.cfg

Search for a string
```bash
:f user
```

Clear the search
```bash
:f
```

Show config options in a pager
```bash
ansible-navigator config -m stdout list
```

### ansible-navigator exec

navigator will bind mount the directory "collections" in the current working directory and install any collections listed there into the execution environment

pull up interactive shell in the execution environment
```bash
ansible-navigator exec
```

### ansible-navigator doc
used like ansible-doc

## Include and import

There are specific docs on docs.ansible.com for each include and import option.

[Include role](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/include_role_module.html)
[Include tasks](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/include_tasks_module.html)
[import role](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/import_role_module.html)
[Import tasks](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/import_tasks_module.html)
[Import Playbooks](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/import_playbook_module.html)


