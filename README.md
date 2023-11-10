# Notes_crt_rh294

### Requirements
There are 18 questions in total.
You will need five RHEL 8 (or CentOS :sunglasses: virtual machines to be able to successfully complete all questions.

### Optional Automatic Exam Setup Available
Here is an automated exam environment deployment for Mac/Linux/Windows that deploys the practice exam environment for you, including IPA server/client installation and configuration. You can also use your own lab environment. Navigate to the respective repo you wish to use for this practice exam and follow the README instructions:

https://github.com/rdbreak/rhce8env,
https://github.com/rdbreak/ansible27env,
https://github.com/rdbreak/ansible8env.

Check the included `README.md` for the environment info and instructions. Reach out to @rdbreak with any questions or concerns.

One VM will be configured as an Ansible control node. Other four VMs will be used to apply playbooks to solve the sample exam questions. The following FQDNs will be used throughout the sample exam.

**control.example.com** – Ansible control node

**node1.example.com** – managed node

**node2.example.com** – managed node

**node3.example.com** – managed node

**node4.example.com** – managed node

There are a couple of requirements that should be met before proceeding further:

**control.example.com** server has password-less SSH access to all managed servers (using the root user).
**node1:4.example.com** servers have a 5GB secondary `/dev/sdb` disk attached.
There is a regular “user” created on all the servers.


### Tips and Suggestions
- I tried to cover as many exam objectives as possible, however, note that there will be no questions related to dynamic inventory.
- Some questions may depend on the outcome of others. Please read all questions before proceeding.


## Sample Exam Questions
*Note - you have root access to all servers.*

#### Task 1: Ansible Installation and Configuration
**(If you’re using the automated deployments, Ansible is already installed)**
Install the ansible package on the control node (including any dependencies) and configure the following:

Create a regular user **automation** with the password of **devops**. Use this user for all sample exam tasks.
All playbooks and other Ansible configuration that you create for this sample exam should be stored in `/home/automation/plays`.
Create a configuration file `/home/automation/plays/ansible.cfg` to meet the following requirements:

1. The roles path should include `/home/automation/plays/roles`, as well as any other path that may be required for the course of the sample exam.
2. The inventory file path is `/home/automation/plays/inventory`.
3. Privileged escalation is **disabled** by default.
4. Ansible should be able to manage **10 hosts** at a single time.
5. Ansible should connect to all managed nodes using the **automation** user.

Create an inventory file `/home/automation/plays/inventory` with the following:

1. node1.ansi.example.com should be a member of the **proxy** host group.
2. node2.ansi.example.com should be a member of the **webservers** host group.
3. node3.ansi.example.com should be a member of the **webservers** and **database** host group.
4. node4.ansi.example.com should be a member of the **database** host group.


#### Task 2: Ad-Hoc Commands
Create an SSH keypair. Write a script `/home/automation/plays/adhoc` that uses Ansible ad-hoc commands to achieve the following:

1. User **automation** is created on all inventory hosts.
2. SSH key (that you generated) is copied to all inventory hosts for the **automation** user and stored in `/home/automation/.ssh/authorized_keys`.
3. The **automation** user is allowed to elevate privileges on all inventory hosts without having to provide a password.
4. After running the ad-hoc script, you should be able to SSH into all inventory hosts using the **automation** user without password, as well as a run all privileged commands.

#### Task 3: File Content
Create a playbook `/home/automation/plays/motd.yml` that runs on all inventory hosts and does the following:

1. The playbook should replace any existing content of `/etc/motd` with text. Text depends on the host group.
2. On hosts in the **proxy** host group the line should be “Welcome to HAProxy server”.
3. On hosts in the **webservers** host group the line should be “Welcome to Apache server”.
4. On hosts in the **database** host group the line should be “Welcome to MySQL server”.


#### Task 4: Configure SSH Server
Create a playbook `/home/automation/plays/sshd.yml` that runs on all inventory hosts and configures SSHD daemon as follows:

1. banner is set to `/etc/motd`
2. X11Forwarding is disabled
3. MaxAuthTries is set to 3


#### Task 5: Ansible Vault
Create Ansible vault file `/home/automation/plays/secret.yml`. Encryption/decryption password is **devops**.
Add the following variables to the vault:

1. **user_password** with value of **devops**
2. **database_password** with value of **devops**
3. Store Ansible vault password in the file `/home/automation/plays/vault_key`.

#### Task 6: Users and Groups
You have been provided with the list of users below.
Use `/home/automation/plays/vars/user_list.yml` file to save this content.

```yaml
---
users:
- username: alice
  uid: 1201
- username: vincent
  uid: 1202
- username: sandy
  uid: 2201
- username: patrick
  uid: 2202
```
Create a playbook `/home/automation/plays/users.yml` that uses the vault file `/home/automation/plays/secret.yml` to achieve the following:

1. Users whose user ID starts with 1 should be created on servers in the **webservers** host group. User password should be used from the **user_password** variable.
2. Users whose user ID starts with 2 should be created on servers in the **database** host group. User password should be used from the **user_password** variable.
3. All users should be members of a supplementary group **wheel**.
4. Shell should be set to `/bin/bash` for all users.
5. Account passwords should use the SHA512 hash format.
6. Each user should have an SSH key uploaded (use the SSH key that you create previously).

After running the playbook, users should be able to SSH into their respective servers without passwords.

#### Task 7: Scheduled Tasks
Create a playbook `/home/automation/plays/regular_tasks.yml` that runs on servers in the **proxy** host group and does the following:

1. A root crontab record is created that runs every hour.
2. The cron job appends the file `/var/log/time.log` with the output from the **date** command.


#### Task 8: Software Repositories
Create a playbook `/home/automation/plays/repository.yml` that runs on all servers and does the following:

1. A YUM repository file is created.
2. The name of the repository is **rpms**.
3. The description of the repository is “This is the main repo file”.
4. Repositories are available to use from http://repo.ansi.example.com/BaseOS and http://repo.ansi.example.com/AppStream


#### Task 9: Create and Work with Roles
Create a role called **sample-mysql** and store it in `/home/automation/plays/roles`. The role should satisfy the following requirements:

1. A primary partition number 1 of size 800MB on device `/dev/sdb` is created.
2. An LVM volume group called `vg_database` is created that uses the primary partition created above.
3. An LVM logical volume called `lv_mysql` is created of size 512MB in the volume group `vg_database`.
4. An XFS filesystem on the logical volume `lv_mysql` is created.
5. Logical volume `lv_mysql` is permanently mounted on `/mnt/mysql_backups`.
6. **mariadb** package is installed.
7. Firewall is configured to allow all incoming traffic on MySQL port TCP 3306.
8. MySQL root user password should be set from the variable **database_password** (see task #5).
9. MySQL server should be started and enabled on boot.
10. MySQL server configuration file is generated from the `my.cnf.j2` Jinja2 template with the following content:
```ini
[mysqld]
bind_address = {{ ansible_default_ipv4.address }}
skip_name_resolve datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

Create a playbook `/home/automation/plays/mysql.yml` that uses the role and runs on hosts in the **database** host group.

#### Task 10: Create and Work with Roles (Some More)
Create a role called **sample-apache** and store it in `/home/automation/plays/roles`. The role should satisfy the following requirements:

1. The **httpd**, **mod_ssl** and **php** packages are installed. Apache service is running and enabled on boot.
2. Firewall is configured to allow all incoming traffic on HTTP port TCP 80 and HTTPS port TCP 443.
3. Apache service should be restarted every time the file `/var/www/html/index.html` is modified.
4. A Jinja2 template file `index.html.j2` is used to create the file `/var/www/html/index.html` with the following content:
```go
The address of the server is: {{ ansible_default_ipv4.address }}
```
`{{ ansible_default_ipv4.address }}` is the IP address of the managed node.
Create a playbook `/home/automation/plays/apache.yml` that uses the role and runs on hosts in the **webservers** host group.

#### Task 11: Download Roles From an Ansible Galaxy and Use Them
Use Ansible Galaxy to download and install **geerlingguy.haproxy** role in `/home/automation/plays/roles`.
Create a playbook `/home/automation/plays/haproxy.yml` that runs on servers in the **proxy** host group and does the following:

1. Use **geerlingguy.haproxy** role to load balance request between hosts in the **webservers** host group.
2. Use **roundrobin** load balancing method.
3. HAProxy backend servers should be configured for HTTP only (port 80).
4. Firewall is configured to allow all incoming traffic on port TCP 80.

If your playbook works, then doing “**curl http://node1.ansi.example.com/**” should return output from the web server (see task #10). Running the command again should return output from the other web server.

#### Task 12: Security
Create a playbook `/home/automation/plays/selinux.yml` that runs on hosts in the **webservers** host group and does the following:

1. Uses the selinux **RHEL system role**.
2. Enables **httpd_can_network_connect** SELinux boolean.
3. The change must survive system reboot.


#### Task 13: Use Conditionals to Control Play Execution
Create a playbook `/home/automation/plays/sysctl.yml` that runs on all inventory hosts and does the following:

1. If a server has more than 512MB of RAM, then parameter **vm.swappiness** is set to 10.
2. If a server has less than 512MB of RAM, then the following error message is displayed: **Server memory less than 512MB**


#### Task 14: Use Archiving
Create a playbook `/home/automation/plays/archive.yml` that runs on hosts in the **database** host group and does the following:

1. A file `/mnt/mysql_backups/database_list.txt` is created that contains the following line: dev,test,qa,prod.
2. A gzip archive of the file `/mnt/mysql_backups/database_list.txt` is created and stored in `/mnt/mysql_backups/archive.gz`.


#### Task 15: Work with Ansible Facts
Create a playbook `/home/automation/plays/facts.yml` that runs on hosts in the **database** host group and does the following:

1. A custom Ansible fact **server_role=mysql** is created that can be retrieved from **ansible_local.custom.sample_exam** when using Ansible setup module.


#### Task 16: Software Packages
Create a playbook `/home/automation/plays/packages.yml` that runs on all inventory hosts and does the following:

1. Installs **tcpdump** and **mailx** packages on hosts in the **proxy** host groups.
2. Installs **lsof** and **mailx** and packages on hosts in the **database** host groups.


#### Task 17: Services
Create a playbook `/home/automation/plays/target.yml` that runs on hosts in the **webservers** host group and does the following:

1. Sets the default boot target to **multi-user**.


#### Task 18. Create and Use Templates to Create Customized Configuration Files
Create a playbook `/home/automation/plays/server_list.yml` that does the following:

1. Playbook uses a Jinja2 template `server_list.j2` to create a file `/etc/server_list.txt` on hosts in the **database** host group.
2. The file `/etc/server_list.txt` is owned by the **automation** user.
3. File permissions are set to **0600**.
4. SELinux file label should be set to **net_conf_t**.
5. The content of the file is a list of FQDNs of all inventory hosts.

After running the playbook, the content of the file `/etc/server_list.txt` should be the following:
```
node1.ansi.example.com node2.ansi.example.com node3.ansi.example.com
```
*Note - If the FQDN of any inventory host changes, re-running the playbook should update the file with the new values.*


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
