# Notes_crt_rh294

## Install

```sh
sudo yum install ansible
yum install rhel-system-roles

mkdir -p /[path]/ansible/roles
chown -R [user]:[user] /[path]/ansible/roles
chmod -r 755 chown -R [user]:[user] /[path]/ansible/roles/*
ansible-galaxy role install -r requirements.yml -p roles

vi /etc/sudoers
```

```/etc/sudoers
[user] ALL=(ALL) NOPASSWD:ALL
```

## Inventory

```inventory
[proxy]
node1.[domain].com

[webservers]
node2.[domain].com
node3.[domain].com

[database]
node4.[domain].com
node5.[domain].com

[public:children]
webservers
proxy
```

## Config

```ansible.cfg
[defaults]
inventory = /[dir_path]/inventory
roles_path = ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/[dir_path]/roles

[privilege_escalation]
become = false
become_ask_pass = false
become_method = sudo
become_user = root
remote_user: root
```
## Ad-Hoc Commands

```sh
#!/usr/bin/bash
ansible all -m yum_repository -a "name=[name] description='[description]' baseurl=[baseurl] enabled=yes gpgcheck=yes gpgkey=[gpgkey url]"
```

## Roles

```requeriments.yml
- name: http-role-gz
  src: https://some.webserver.example.com/files/main.tar.gz
```

## Playbook

```playbook.yml
- name: Example playbook all elements
  hosts: all
  gather_facts: true
  vars_files:
    - ./vars/vault.yml

  tasks:
  - name: Install required packages
    yum:
      name:
      - httpd
      - firewalld
      state: present
  - name: Allow required ports
    firewalld:
      permanent: true
      state: enabled
      port: "{{ item }}"
      immediate: true
    with_items:
    - 80/tcp
    - 443/tcp
  - name: Ensure that services are started on boot
    service:
      name: "{{ item }}"
      state: started
      enabled: true
    with_items:
    - httpd
    - firewalld
  - name: Prepare index page
    copy:
      content: "Welcome, you have connected to {{ ansible_facts.fqdn }}\n"
      dest: /var/www/html/index.html
  
  - name: Create users for webservers
    user:
      name: "{{ item.username }}"
      uid: "{{ item.uid }}"
      groups: ['wheel']
      shell: /usr/bin/bash
      password: "{{ user_password | password_hash('sha512', 'password') }}


    - name: Create a logical volume of 512m with disks /dev/sda and /dev/sdb
      community.general.lvol:
        vg: firefly
        lv: test
        size: 1500
        pvs: /dev/vdb
      loop: "{{ ansible_mounts }}"
      when: item.mount == "[volume]" and item.size_available >= 15000
      ignore_errors: yes
  
    - name: Ensure XFS Filesystem exists on each LV
        filesystem:
          dev: "/dev/{{ item.vgroup }}/{{ item.name }}"
          fstype: xfs
        loop: "{{ logical_volumes }}"
```

# Vault

```sh
ansible-vault create vault.yml
ansible-vault encrypt vault.yml
ansible-vault decrypt vault.yml
ansible-vault edit vault.yml
ansible-vault rekey vault.yml
ansible-playbook --vault-password-file=vault-pw-file vault.yml
ansible-playbook --ask-vault-pass vault.yml
```
# host template

```hosts
127.0.0.1 localhost {{ ansible_hostname }} {{ ansible_fqdn }}
127.0.1.1 localhost

{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```

### Cron

```task.yml
- name: remove tempuser.
  at:
    command: userdel -r tempuser
    count: 20
    units: minutes
    unique: yes
- cron:
    name: "Flush Bolt"
    user: "root"
    minute: 45
    hour: 11
    job: "php ./app/nut cache:clear"
```

=========

```notas

# When

ansible_machine == "x86_64"
max_memory == 512
min_memory < 128
min_memory > 256
min_memory <= 256
min_memory >= 512
min_memory != 512
min_memory is defined
min_memory is not defined
memory_available
not memory_available
ansible_distribution in supported_distros

# Ansible facts

Short host name	                            ansible_facts['hostname']
Fully qualified domain name	                ansible_facts['fqdn']
Main IPv4 address (based on routing)	      ansible_facts['default_ipv4']['address']
List of the names of all network interfaces	ansible_facts['interfaces']
Size of the /dev/vda1 disk partition	      ansible_facts['devices']['vda']['partitions']['vda1']['size']
List of DNS servers	                        ansible_facts['dns']['nameservers']
Version of the currently running kernel	    ansible_facts['kernel']
```

```Template.yml
- name: Example playbook all elements
    hosts: database
    gather_facts: true
    tasks:
      - name: Conditions
        block:
          - name: tasks one
            shell:
              cmd: ""
        rescue:
          - name: ""
            shell:
              cmd: ""
        always:
          - name: ""
            shell:
              cmd: ""
        hosts: all

  pre_tasks:
    - name: ""
  roles:
    - role1
  tasks:
    - name: ""
      notify: ""
  post_tasks:
    - name:
      notify: ""
  handlers:
    - name: ""
``` 
