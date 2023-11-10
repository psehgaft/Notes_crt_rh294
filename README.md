# Notes_crt_rh294

# Validate Ansible

Control Nodes
```sh
yum list installed platform-python

subscription-manager register
subscription-manager repos --enable ansible-2-for-rhel-8-x86_64-rpms
yum install ansible

# Validate
ansible --version
ansible -m setup localhost | grep ansible_python_version
```

Managed Hosts

```sh
yum module install python36
```
Config Files order

1 ./ansible.cfg
2 ~/.ansible.cfg
3 /etc/ansible/ansible.cfg

```ansible.cfg
[defaults]
inventory = ./inventory
remote_user = user
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```sh

Static inventory

```sh
sudo vim /etc/ansible/hosts
```

```yml
[nodes]
node-[1:10].server.com

[db]
db-[99:101].server.com

[webserver]
node-[a:b].server.com

[tecnology:children]
webserver
db
```

Validate inventory

```sh
ansible all -i inventory --list-hosts
ansible ungrouped -i inventory --list-hosts
ansible [group] -i inventory --list-hosts
```

Coppy SSH

```sh
ssh-copy-id root@[node-hostname]

/etc/sudoers.d
.. user ALL=(ALL) NOPASSWD:ALL
```

Execute playbooks

```sh
# module
ansible host-pattern -m module [-a 'module arguments'] [-i inventory] -o

# playbook
ansible-playbook -i inventory {or} [servers-scoupe] playbook.yml

#Test
ansible-playbook -i inventory {or} [servers-scoupe] -C playbook.yml
```

Validate module

```sh
ansible-doc -s [module]
```

Validate playbook syntax

```sh
ansible-playbook --syntax-check playbook.yml
```

## Proyect structure

```ls
project
├── ansible.cfg
├── group_vars
│   ├── datacenters
│   ├── datacenters1
│   └── datacenters2
├── host_vars
│   ├── host1
│   ├── host2
│   ├── host3
├── inventory
└── playbook.yml
│   ├── host3
├── roles
│   ├── role1
```
