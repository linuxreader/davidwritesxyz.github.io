---
title: 'Including and importing Files'
description: 'Including and importing Files'
showDate: false
---

include
- Dynamically processed at the moment that Ansible reaches that content.   

import
- Ansible performs the import operation before starting to work on the tasks in the playbook.

Files can be included and imported at different levels:
- Roles 
- Playbooks
	- Can be imported as a complete playbook.
	- Cannot do this from within a play.
	- Can be imported only at the top level of the playbook.
- Tasks
	- A task file imported or included in another task.
- Variables 
	- Variables can be maintained in external files and included in a playbook. 
	- makes managing generic multipurpose variables easier.

#### Importing Playbooks

- Common in a setup where one master playbook is used, from which different additional playbooks are included. 
- The master playbook could have the name site.yaml, and it can be used to include playbooks for each specific set of servers, for instance. 
- When a playbook is imported, this replaces the entire play. 
- Cannot import a playbook at a task level; it needs to happen at a play level. 

Playbook to Be Imported
```yaml
    - hosts: all
      tasks:
      - debug:
          msg: running the imported play
```

Importing a Playbook
```yaml
    ---
    - name: run a task
      hosts: all
      tasks:
      - debug:
          msg: running task1
    
    - name: importing a playbook
      import_playbook: listing104.yaml
```

#### Importing and Including Task Files

**import_tasks**
- Statically imported while executing the playbook. 
- Cannot be used with loops
- If a variable is used to specify the name of the file to import, this cannot be a host or group inventory variable.
- When you use a **when** statement on the entire **import_tasks** file, the conditional statements are applied to each task that is involved.

**include_tasks**
- Dynamically included at the moment they are needed. 
- Dynamically including task files is recommended when the task file is used in a conditional statement. 
- If task files are mainly used to make development easier by working with separate task files, they can be statically imported.
- When you use the `ansible-playbook --list-tasks` command, tasks that are in the included tasks are not displayed.
- Cannot use `ansible-playbook --start-at-task` to start a playbook on a task that comes from an included task file.
- Cannot use a **notify** statement in the main playbook to trigger a handler that is in the included tasks file.


When you use includes and imports to work with task files, the recommendation is to store the task files in a separate directory. Doing
so makes it easier to delegate task management to specific users.

#### Using Variables When Importing and Including Files

- imported and included files should be as generic as possible. 
- Define specific items as variables.  
```yaml
    - name: install {{ package }}
      yum:
        name: "{{ package }}"
        state: latest
    - name: start {{ service }}
      service:
        name: "{{ service }}"
        enabled: true
        state: started
```

```yaml
    - name: install the firewall
      package:
        name: "{{ firewall_package }}"
        state: latest
    - name: start the firewall
      service:
        name: "{{ firewall_service }}"
        enabled: true
        state: started
    - name: open the port for the service
      firewalld:
        service: "{{ item }}"
        immediate: true
        permanent: true
        state: enabled
      loop: "{{ firewall_rules }}"
```

```yaml
    ---
    - name: setup a service
      hosts: ansible2
      tasks:
      - name: include the services task file
        include_tasks: tasks/service.yaml
        vars:
          package: httpd
          service: httpd
        when: ansible_facts[’os_family’] == ’RedHat’
      - name: import the firewall file
        import_tasks: tasks/firewall.yaml
        vars:
          firewall_package: firewalld
          firewall_service: firewalld
          firewall_rules:
          - http
          - https
```


### Lab: Using Includes and Imports

Create a simple master playbook that installs a service. The name of the service is defined in a variable file, and the
specific tasks are included through task files.

1\. Open the file exercise103-vars.yaml and define three variables as
follows:

``` pre1
packagename: vsftpd
servicename: vsftpd
firewalld_servicename: ftp
```

2\. Create the exercise103-ftp.yaml file and give it the following
contents to install, enable, and start the vsftpd service and also to
make it accessible in the firewall:

``` pre1
- name: install {{ packagename }}
  yum:
    name: "{{ packagename }}"
    state: latest
- name: enable and start {{ servicename }}
  service:
    name: "{{ servicename }}"
    state: started
    enabled: true
- name: open the service in the firewall
  firewalld:
    service: "{{ firewalld_servicename }}"
    permanent: yes
    state: enabled
```

3\. Create the exercise103-copy.yaml file that manages the
/var/ftp/pub/README file and make sure it has the following contents:

``` pre1
- name: copy a file
  copy:
    content: "welcome to this server"
    dest: /var/ftp/pub/README
```

4\. Create the master playbook exercise103.yaml that includes all of
them and give it the following contents:

``` pre1
---
- name: install vsftpd on ansible2
  vars_files: exercise103-vars.yaml
  hosts: ansible2
  tasks:
  - name: install and enable vsftpd
    import_tasks: exercise103-ftp.yaml
  - name: copy the README file
    import_tasks: exercise103-copy.yaml
```

5\. Run the playbook and verify its output

6\. Run an ad hoc command to verify the /var/ftp/pub/README file has
been created: **ansible ansible2 -a "cat /var/ftp/pub/README"**.


### End-of-Chapter Lab

In the end-of-chapter lab with this chapter, you reorganize a playbook
to work with several different files instead of one big file. Do this
according to the instructions in Lab 10-1.

### Lab 10-1

The lab82.yaml file, which you can find in the GitHub repository that
goes with this course, is an optimal candidate for optimization.
Optimize this playbook according to the following requirements:

• Use includes and import to make this a modular playbook where
different files are used to distinguish between the different tasks.

• Optimize this playbook such that it will run on no more than two hosts
at the same time and completes the entire playbook on these two hosts
before continuing with the next host.