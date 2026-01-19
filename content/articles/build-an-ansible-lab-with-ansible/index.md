---
title: 'Building local Virtual Machines with Ansible and LibVirt'
description: 'How to build local virtual machines using Ansible and Libvirt'
draft: true
---

- Set up two Alma Linux 10 virtual machines using ansible and libvirt

Download kvm_provision role
Add variables
Add VM template
add tasks
create playbook
provisioning script
set up a dhcp reservation

When I started studying for RHCE, I was spending tons of time deploying and destroying virtual machines. 

I thought.. 

Why not start my automation journey right? I had four goals:
1. Create a Virtual Machine using an Alma 10 Cloud init image. 
2. Have RAM, Storage, etc. defined in a file. 
3. Set up the root password and ssh keys.
4. Assign an IP address.

This guide uses [Libvirt](https://libvirt.org/index.html) to manage [KVM/QEMU](https://www.qemu.org/) Virtual Machines and the Ansible deploy them. 

## Creating the role

First, Initialize the role and remove unused directories:  
`cd roles && ansible-galaxy role init kvm_provision && cd kvm_provision/ && rm -r files handlers vars`

## Defining role variables

Add these default variables to main.yml. You can to a web search for cloud images if you want a different OS and sha. Here, you also define hardware specs, root password information, and ip addressing. 

We'll override some of this in our deployment script later:

`cat defaults/main.yml`

```yaml
---
base_image_name: AlmaLinux-10-GenericCloud-latest.x86_64.qcow2
base_image_url: https://repo.almalinux.org/almalinux/10/cloud/x86_64/images/{{ base_image_name }}
base_image_sha: 8729862a9e2d93c478c2334b33d01d7ee21934a795a9b68550cc1c63ffd0efae 
libvirt_pool_dir: "/var/lib/libvirt/images"
vm_name: Alma10-dev
vm_vcpus: 2
vm_ram_mb: 2048
vm_net: default
vm_root_pass: test123
cleanup_tmp: no
ssh_key: /root/.ssh/id_rsa.pub
ip_addr: 192.168.122.250
gw_addr: 192.168.122.1
vm_mac: 52:54:00:a0:b0:00
disk_size: 20
```

## Defining a VM template

The [community.libvirt.virt ](https://docs.ansible.com/projects/ansible/latest/collections/community/libvirt/virt_module.html)Ansible module will be used to provision our KVM Virtual Machine. This module uses a VM definition file in XML format with [libvirt syntax](https://libvirt.org/formatdomain.html). You can get a template from a current VM and edit from there. 

The power of using a [Jinja template](https://www.davidwrites.xyz/notes/rhce-notes/jinja2templates/) is that we can insert variables to the template for re-usability. 

```bash
cat templates/vm-template.xml.j2 
```

```xml
<domain type='kvm'>
  <name>{{ vm_name }}</name>
  <memory unit='MiB'>{{ vm_ram_mb }}</memory>
  <vcpu placement='static'>{{ vm_vcpus }}</vcpu>
  <os>
    <type arch='x86_64' machine='pc-q35-5.2'>hvm</type>
    <boot dev='hd'/>
  </os>
  <cpu mode='host-model' check='none'/>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='{{ libvirt_pool_dir }}/{{ vm_name }}.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x05' slot='0x00' function='0x0'/>
      <!-- Specify the disk size using a variable -->
      <size unit='GiB'>{{ disk_size }}</size>
    </disk>
    <interface type='network'>
      <source network='{{ vm_net }}'/>
      <model type='virtio'/>
      <target dev='eth0'/>
      <!-- Specify the mac address using a variable -->
      <mac address="{{ vm_mac }}" />
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
    <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <input type='tablet' bus='usb'>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='spice' autoport='yes'>
      <listen type='address'/>
      <image compression='off'/>
    </graphics>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </video>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x06' slot='0x00' function='0x0'/>
    </memballoon>
    <rng model='virtio'>
      <backend model='random'>/dev/urandom</backend>
      <address type='pci' domain='0x0000' bus='0x07' slot='0x00' function='0x0'/>
    </rng>
  </devices>
</domain>
```

## Defining role tasks

Here is the task list for this playbook. 

```yaml
---
# Commented out on my setup. Using an immutable distro with this stuff already set up.
#- name: Requirements
#  package:
#    name:
#      - guestfs-tools
#      - python3-libvirt
#    state: present
#  become: yes

- name: List existing VMS
  community.libvirt.virt:
    command: list_vms
  register: existing_vms
  changed_when: no

- name: Create VM if its does not exist
  block:
  - name: Download base image
    get_url:
      url: "{{ base_image_url }}"
      dest: "/tmp/{{ base_image_name }}"
      force: yes
      checksum: "sha256:{{ base_image_sha }}"
      
  - name: Copy base image to libvirt directory
    copy:
      dest: "{{ libvirt_pool_dir }}/{{ vm_name }}.qcow2"
      src: "/tmp/{{ base_image_name }}"
      force: no
      remote_src: yes 
      mode: 0660
    register: copy_results

  - name: Configure the image
    command: |
      virt-customize -a {{ libvirt_pool_dir }}/{{ vm_name }}.qcow2 \
      --hostname {{ vm_name }} \
      --root-password password:{{ vm_root_pass }} \
      --ssh-inject 'root:file:{{ ssh_key }}' \
      --uninstall cloud-init 
    # Compatibility options for Fedora BlueFin, not needed on non-immutable distros.
    register: image_result  
    failed_when:
      - image_result.rc != 0
      - "'SELinux relabelling' not in image_result.stdout"

# Use the xml template to define the vm settings.
  - name: Define vm
    community.libvirt.virt:
      command: define
      xml: "{{ lookup('template', 'vm-template.xml.j2') }}"
    when: "vm_name not in existing_vms.list_vms"

  - name: Ensure VM is started
    community.libvirt.virt:
      name: "{{ vm_name }}"
      state: running
    register: vm_start_results
    until: "vm_start_results is success"
    retries: 15
    delay: 2

  - name: Ensure temporary file is deleted
    file:
      path: "/tmp/{{ base_image_name }}"
      state: absent
    when: cleanup_tmp | bool
```

## Creating the playbook
Changed my user to own the libvirt directory:
`chown -R david:david /var/lib/libvirt/images`

Create playbook `kvm_provision.yaml`
```yml
---
# Example useage: ansible-playbook kvm_provision.yaml -K -e vm=testvm-01 -e ip_addr=192.168.122.3 -e disk_size=31 -e vm_mac=52:54:00:a0:b0:01

- name: Deploy VM based on cloud image
  hosts: localhost
  gather_facts: yes
  become: yes
  vars:
    pool_dir: "/var/lib/libvirt/images"
    cleanup: no
    net: default
    ssh_pub_key: "/var/home/david/.ssh/id_ed25519.pub"
    ansible_password: "{{ ansible_user_pass }}"

  tasks:
    - name: Get VMs list
      community.libvirt.virt:
        command: list_vms
      register: existing_vms
      changed_when: no

    - name: KVM Provision role
      include_role:
        name: kvm_provision
      vars:
        libvirt_pool_dir: "{{ pool_dir }}"
        vm_name: "{{ vm }}"
        vm_vcpus: "{{ vcpus }}"
        vm_ram_mb: "{{ ram_mb }}"
        vm_net: "{{ net }}"
        cleanup_tmp: "{{ cleanup }}"
        ssh_key: "{{ ssh_pub_key }}"
      when: "vm_name not in existing_vms.list_vms"
```

Add the libvirt collection
`ansible-galaxy collection install community.libvirt`

Create a VM with a new name
`ansible-playbook -K kvm_provision.yaml -e vm=ansible1`



```
 --run-command 'useradd -m -p {{ ansible_user }} ; chage -d 0 {{ ansible_user }} ; cat {{ ansible_password }} > passwd {{ ansible_user }} --stdin"'
 
  - name: Add ssh keys
    command: |
      virt-customize -a {{ libvirt_pool_dir }}/{{ vm_name }}.qcow2 \
      --ssh-inject '{{ ansible_user }}:file:{{ ssh_key }}'

```

## Assigning static IP 
https://www.cyberciti.biz/faq/linux-kvm-libvirt-dnsmasq-dhcp-static-ip-address-configuration-for-guest-os/