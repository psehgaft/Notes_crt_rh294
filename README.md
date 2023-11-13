# Notes_crt_rh294

## Scripts

### Install

```sh
sudo yum install ansible
yum module install python36

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

  - name: web server is enabled
      service:
        name: httpd
        enabled: true

```
