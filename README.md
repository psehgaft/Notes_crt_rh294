# Notes_crt_rh294

## Scripts

### Install

```sh
sudo yum install ansible
yum module install python36
yum install rhel-system-roles
mkdir -p /[path]/ansible/roles
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install [community.role]

vi /etc/sudoers
```

```/etc/sudoers

[user] (ALL) NOPASSWD: ALL

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
log_path = /var/log/ansible/execution.log
roles_path = ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/[dir_path]/roles

[privilege_escalation]
become = false
become_ask_pass = false
become_method = sudo
become_user = root
remote_user: root
```
## Repos

```sh
ansible all -m yum_repository -a "name=[name] description='[description]' baseurl=[baseurl] enabled=yes gpgcheck=yes gpgkey=[gpgkey url]"
```

## Facts

```/data-facts/custom.fact
[general]
package = httpd
service = httpd
state = started
enabled = true
```

## Roles

```requeriments.yml
# from a webserver, where the role is packaged in a tar.gz
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

  - name: server.example.com in /etc/hosts
      template:
        src: ../templates/hosts.j2
        dest: /etc/ottherhosts
        # line: "{{ ansible_facts.default_ipv4.address }} {{ ansible_facts.fqdn }} {{ ansible_facts.hostname }}"
        # state: present
        owner: root
        group: root
        mode: 0644
    vars:
      hostname1: "{{ ansible_facts.fqdn }}"
      ip1: "{{ ansible_facts.default_ipv4.address }}"
      host: "{{ ansible_facts.hostname }}"

  - name: Users exist and are in the correct groups
    user:
      name: "{{ item.name }}"
      state: present
      groups: "{{ item.groups }}"
    loop:
      - name: jane
        groups: wheel
      - name: joe
        groups: root

- name: Example playbook all elements
  hosts: webservers
  gather_facts: true
  tasks:

  - name: httpd package is present
      yum:
        name:
          - "{{ ansible_facts.ansible_local.custom.general.service }}"
          - php
        state: latest

  - name: correct index.html is present
      copy:
        content: "Welcome {{ ansible_facts.default_ipv4.address }} {{ ansible_facts.fqdn }}\n"
        dest: /var/www/html/index.html
   with_items: "{{ groups['dev'] }}"
   notify:
      - restart apache

  - name: firewalld enabled and running
      service:
        name: firewalld
        enabled: true
        state: started

  - name: firewalld permits access to httpd service
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes

  - name: httpd is started
      service:
        name: httpd
        state: started
        enabled: true

  handlers:

  - name: restart apache
    service:
      name: httpd
      state: restarted

- name: Example playbook all elements
  hosts: proxy
  tasks:

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
        - name: revert tasks one
          shell:
            cmd: ""
      always:
        - name: tasks alwayss
          shell:
            cmd: ""

  - name: httpd package is present
      yum:
        name:
          - mariadb
        state: latest

  - name: mariadb enabled and running
      service:
        name: mariadb
        enabled: true
        state: started

  - name: Create a logical volume of 512m with disks /dev/sda and /dev/sdb
    community.general.lvol:
      vg: firefly
      lv: test
      size: 1500
      pvs: /dev/vdb
    loop: "{{ ansible_mounts }}"
    when: item.mount == "[volume]" and item.size_available >= 15000
    ignore_errors: yes

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
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```

## Sintax

```play.yml
- name: Play sintaxis
  hosts: all
  pre_tasks:
    - name:
  roles:
    - role1
  tasks:
    - name:
      notify: my handler
  post_tasks:
    - name:
      notify: my handler
  handlers:
    - name: 
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
```

