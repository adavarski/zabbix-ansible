# Zabbix + Ansible Installation & Setup 

![Badge](https://img.shields.io/badge/ansible-2.9.10-blue)

## Zabbix Server : Supported OS

- Ubuntu20
- Ubuntu22
- Debian11
- Rocky8
- Almalinux8
- Centos8

### Zabbix supported versions

|   Zabbix       |  Debian 11 | Ubuntu 20 | Ubuntu 22 | Rocky 8 | Almalinux 8 | Centos 8 |
|   ----         |     ---    |    ---    |    ---    |   ---   |     ---     |    ---   |
|    4.4         |      No    |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    5.0         |      Yes   |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    5.2         |      Yes   |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    5.4         |      Yes   |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    6.0         |      Yes   |     Yes   |      Yes  |   Yes   |       Yes   |    Yes   |

DB: MySQL or Postgresql

To limit the use of disk space by the mysql binlog log file, which by default is 30 days for purge, you can reduce the database with the command below:
```
show variables like '%expire_logs%';
SET GLOBAL binlog_expire_logs_seconds = (60*60*24*10);
```

### Variables
| Nome | Descrição | Default | 
|------|-----------|---------|
| zabbix_version | zabbix-server version| 4.4|
| postgresql_version | postgresql version | 13 |
| zabbix_server_database_long |  database[mysql/pgsql] |  mysql
| zabbix_server_database | database[mysql/pgsql] | mysql
| zbx_database_address | IP database | 127.0.0.1
| zbx_front_address | IP front | 127.0.0.1
| zbx_server_address | IP zabbix | 127.0.0.1
| zbx_server_ha | IP zabbix node 2 (6.0 >) | 127.0.0.1
| zabbix_server_ha | HA (6.0 >) enable|disable | disable

### Example localhost Mysql (DEFAULT)
```yaml
---
- hosts: all
  become: true
  roles:
    - {role: roles/mysql}
    - {role: roles/zabbix-server}
    - {role: roles/zabbix-front}
```
### Example localhost postgresql
```yaml
---
- hosts: all
  become: true
  vars:
    zabbix_server_database: pgsql
    zabbix_server_database_long: pgsql
  roles:
    - {role: roles/postgresql}
    - {role: roles/zabbix-server}
    - {role: roles/zabbix-front}

```
### Example localhost postgresql version 
```yaml
---
- hosts: all
  become: true
  vars:
    zabbix_server_database: pgsql
    zabbix_server_database_long: pgsql
    postgresql_version: 14
  roles:
    - {role: roles/postgresql}
    - {role: roles/zabbix-server}
    - {role: roles/zabbix-front}

```
### Example MySQL on separate server
```yaml
---
- name: Install mysql
  hosts: db
  vars:
    zbx_server_address: IP-SERVER-ZABBIX
    zbx_front_address: IP-FRONT
    zbx_server_ip_ha: IP-SERVER-NODE-2 ##(HA 6.0)
  become: yes
  roles:
  - mysql

- name: Install Zabbix Server
  hosts: zbx
  vars:
    zbx_database_address: IP-SERVER-DATABASE
    zabbix_server_ha: enable ##(HA 6.0)
  become: yes
  roles:
  - zabbix-server

- name: Install Front
  hosts: web
  vars:    
    zabbix_server_ha: enable ##(HA 6.0)
    zbx_database_address: IP-SERVER-DATABASE
    zbx_server_address: IP-SERVER-ZABBIX
  become: yes
  roles:
  - zabbix-front
```
### Example postgresql
```yaml
---
- name: Install postgresql
  hosts: db
  vars:
    zbx_server_address: IP-SERVER-ZABBIX
    zbx_front_address: IP-SERVER-FRONT
    zbx_server_ip_ha: IP-SERVER-NODE-2 ##(HA 6.0)
  become: true
  roles:
  - postgresql

- name: Install Zabbix Server
  hosts: zabbix
  vars:
    zabbix_server_ha: enable ##(HA 6.0)
    zbx_database_address: IP-SERVER-DATABASE
    zabbix_server_database_long: pgsql
    zabbix_server_database: pgsql
  become: true
  roles:
  - zabbix-server

- name: Install Front
  hosts: web
  vars:
    zabbix_server_ha: enable ##(HA 6.0)
    zbx_server_address: IP-SERVER-ZABBIX
    zbx_database_address: IP-SERVER-DATABASE
    zabbix_server_database: pgsql
  become: true
  roles:
  - zabbix-front
```
### HA 6.0 +
```yaml
[db]
IP-DATABASE ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
[zabbix]
IP-ZABBIX_SERVER ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
IP-ZABBIX_SERVER-NODE2 ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
[web]
IP_FRONT ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
```

### Execute playbook
```
ansible-playbook -i hosts playbook-zabbix-server.yml --extra-vars "zabbix_version=6.0"
```

## Zabbix-proxy via ansible 
==============================

### Requerimentos
------------
- ansible 2.9.6
- Tested on Ubuntu20, Rocky8, Centos7 

### Zabbix version
--------------
To be set in playbook.yml file according to default zabbix-server version 4.4

## Variables

| Nome | Descrição | Default | 
|------|-----------|---------|
| zabbix_version | zabbix-server | 4.4|
| zabbix_server_database_long | database[mysql/sqlite3] | sqlite3
| zabbix_server_database | Tipo de database[mysql/sqlite3] | sqlite3
| zabbix_server_ip | ip zabbix-server | 127.0.0.1

```
mysql_databases:
  - name: "{{ zabbix_proxy_dbname }}"
    login_password: "{{ mysql_root_pass }}"
    encoding: utf8
    collation: utf8_bin
mysql_users:
  - login_password: "{{ mysql_root_pass }}"
    name: "{{ zbx_database_user }}"
    password: "{{ mysql_zabbix_pass.stdout }}"
    priv: "{{ zabbix_proxy_dbname }}.*:ALL"
mysql_root_users:
  - login_user: root
    login_password: "" 
    name: "{{ mysql_root_username }}"
    password: "{{ mysql_root_pass }}"
    host: "{{ zbx_user_privileges }}"
    priv: "*.*:ALL,GRANT"
```
Important
-----------------
There are 2 database_types that will be supported: mysql and sqlite. You will need to enter the variables in the playbook or extra-vars for the database you want to use. Otherwise, it will follow the default, which is SQlite3

### Playbook Example
----------------
```
---
- hosts: all
  vars:
    zabbix_server_ip: 'IP-ZABBIX-SERVER'
    zabbix_version: 4.4
    zabbix_proxy_database: mysql or sqlite3 
    zabbix_proxy_database_long: mysql or sqlite3
  roles:
    - zabbix
  become: yes
  
```
### Invenary
--------------
```
[zabbix-proxy]
xx.xx.xx.xx ansible_ssh_private_key_file=/Path-Dir/key.pem ansible_user=vagrant
```

## Zabbix-agent supported OS:

- Amazon1/2
- CentOS6
- CentOS7
- CentOs8
- Rocky8
- Debian10
- Debian11 
- Ubuntu18
- Ubuntu20
- Ubuntu22

### 6.0 related

### Variables

| Nome | Descrição | Default | 
|------|-----------|---------|
| zabbix_version | zabbix-server | 4.4|
| zabbix_hostname | Hostname zabbix-agent | "{{ ansible_hostname }}" |
| zabbix_hostmetadata | auto-registration | os-linux |
| zabbix_agent2_update | agent2 | False |
| zabbix_agent2_install | zabbix-agent2  **version 5.0**  | False | 
| zabbix_agent_update | zabbix-agent 1 | False |
| zabbix_agent_install | zabbix-agent 1 | False | 
| zabbix_server_ip | IP zabbix-server | 127.0.0.1| 
| porta_agent2 | port agent2 | 10052 |
| porta_agent | port agent | 10050 |

### Important
 One or both of zabbix_agent_install(DEFAULT) and zabbix_agent2_install variables must be selected changing to True for the installation process to take place, entering both will install both types on different ports if you want to update only agent version, use zabbix_agent_updat or zabbix_agent2_update variables:
  
  - 10050: **zabbix-agent** 
  - 10052: **zabbix-agent2**  
 
### Inventory

### Playbook example
```yaml
---
- name: Install Zabbix-agent
  hosts: all
  vars:
    zabbix_server_ip: 'IP-SERVER'
    zabbix_version: 4.4
    zabbix_agent_install ou zabbix_agent2_install: True
    ### Update variables only when updating
    zabbix_agent_update ou zabbix_update2_install: True
  become: true
  roles:
  - zabbix-agent
```
### Playbook
``` 
ansible-playbook -i hosts playbook.yml --extra-vars "zabbix_agent_install=True zabbix_version=5.0 zabbix_server_ip=127.0.0.1"

ansible-playbook -i hosts playbook.yml --extra-vars "zabbix_agent_update=True zabbix_version=5.4 zabbix_server_ip=127.0.0.1"
``` 



## Local installation and testing with Vagrant & VirtualBox

Pre: Install Vagrant & VirtualBox
- https://www.vagrantup.com/downloads
- https://www.virtualbox.org/wiki/Downloads

Vagranfile -> almalinux 8

```ruby
vms = {
#'rocky-srv' => {'memory' => '2024', 'cpus' => '1', 'ip' => '11', 'box' => 'rockylinux/8'},
'almalinux-srv' => {'memory' => '1024', 'cpus' => '1', 'ip' => '12', 'box' => 'almalinux/8'},
#'debian-srv' => {'memory' => '1024', 'cpus' => '1', 'ip' => '13', 'box' => 'debian/buster64'},
#'ubuntu-srv' => {'memory' => '1024', 'cpus' => '2', 'ip' => '14', 'box' => 'ubuntu/focal64'},
}
```
### Provisioning (vagrant commands)
```
vagrant up 
vagrant ssh 
vagrant halt 
vagrant destroy -f
```

### Add this line to /etc/hosts
```
192.168.33.12 almalinux-srv.example.com
```

http://192.168.33.12 or http://almalinux-srv.example.com -> Credentials (default) : Admin/zabbix

```
$ vagrant ssh

[root@almalinux-srv etc]# pwd
/etc

[root@almalinux-srv etc]# diff zabbix_server.conf zabbix_server.conf.ORIG
12c12
< ListenPort=10051
---
> # ListenPort=10051
89c89
< DBHost=localhost
---
> # DBHost=localhost
125c125
< DBPassword=Tg0z64OVNzFwNA==
---
> # DBPassword=

[root@almalinux-srv etc]# systemctl restart zabbix-server
[root@almalinux-srv etc]# systemctl status zabbix-server

### Install zabbix-agent on Vagrant host
$ ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.1.100
$ ansible-playbook -i inventory -l zabbix-agents playbook-zabbix-agent.yml --extra-vars "zabbix_agent_update=True zabbix_version=6.0 zabbix_server_ip=192.168.33.12"
$ diff /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.ORIG
102c102
< Server=192.168.33.12, 127.0.0.1, 192.168.1.100
---
> Server=192.168.33.12
118c118
< ListenIP=0.0.0.0
---
> # ListenIP=0.0.0.0
402c402
< # TLSPSKFile=
---
> # TLSPSKFile=
\ No newline at end of file

$ sudo mkdir /etc/zabbix/zabbix_agentd.d
$ sudo systemctl restart zabbix-agent
$ sudo systemctl status zabbix-agent
```

Screenshots: 
<img src="pictures/Zabbix-Configuration-Add-host..png?raw=true" width="900">
<img src="pictures/Zabbix-hosts.png?raw=true" width="900">
<img src="pictures/Zabbix-node1-system-performance.png?raw=true" width="900">

