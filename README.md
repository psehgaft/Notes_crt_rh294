# Notes_crt_rh294

## Inventory

```inventory
[proxy]
managed1.example.com

[webservers]
managed2.example.com
managed3.example.com

[database]
managed3.example.com
managed4.example.com

[public:children]
webservers
proxy
```

## Config

```cfg
[defaults]
inventory = ./inventory
forks = 8
log_path = /var/log/ansible/execution.log
roles_path = ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/home/automation/plays/roles

[privilege_escalation]
become = false
become_ask_pass = false
become_method = sudo
```
