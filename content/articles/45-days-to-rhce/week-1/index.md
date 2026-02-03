---
draft: true
---

## Ad Hoc Commands
https://docs.ansible.com/projects/ansible/latest/command_guide/intro_adhoc.html

```bash
man ansible
```

Ad Hoc commands are not a complicated topic. You can use them to gather information from a server or to make changes. Though you probably don't want to make undocumented changes from an indempotent standpoint.

These are useful for scripts. Like this one that provisions new VMS and brings them under the control of our control node:  
```bash
#!/bin/bash
ansible-playbook kvm_provision.yaml -e vm=podman-01 -e ip_addr=192.168.122.3 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:01 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=podman-01   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=podman-02 -e ip_addr=192.168.122.4 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:02 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=podman-02   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=ansible1 -e ip_addr=192.168.122.5 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:03 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=ansible1   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=ansible2 -e ip_addr=192.168.122.6 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:04 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=ansible2   -e ansible_user=root

ansible-playbook kvm_provision.yaml -e vm=ansible3 -e ip_addr=192.168.122.7 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:05 &&
ansible-playbook playbooks/bootstrap.yaml   -e target=ansible3   -e ansible_user=root
```

This may also prove useful and is linked to from the Documentation page for Ad Hoc commands: [patterns](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_patterns.html#intro-patterns)

