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
