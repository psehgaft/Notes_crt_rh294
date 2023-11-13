# Notes_crt_rh294

## Scripts

### Install

```sh
sudo yum install ansible
yum module install python36
mkdir -p /[path]/ansible/roles

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

## Playbook

```playbook.yml
- name: Example playbook all elements
  hosts: all
  tasks:

  - name: server.example.com in /etc/hosts
      lineinfile:
        path: /etc/ottherhosts
        line: '192.0.2.42 server.example.com server'
        state: present

- name: Example playbook all elements
  hosts: webservers
  tasks:

  - name: httpd package is present
      yum:
        name:
          - httpd
          - php
        state: latest

  - name: correct index.html is present
      template:
        src: ../templates/index.html.j2
        dest: /var/www/html/index.html

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

- name: Example playbook all elements
  hosts: proxy
  tasks:

- name: Example playbook all elements
  hosts: database
  tasks:

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

```

# Vault


```sh
ansible-vault create vault.yml
ansible-vault encrypt vault.yml
ansible-vault decrypt vault.yml
ansible-vault rekey vault.yml

ansible-playbook --vault-password-file=vault-pw-file vault.yml

ansible-playbook --ask-vault-pass vault.yml
```