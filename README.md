# Zabbix ansible install

![Badge](https://img.shields.io/badge/ansible-2.9.10-blue)

## Supported OS

- Ubuntu20
- Ubuntu22
- Debian11
- Rocky8
- Almalinux8
- Centos8

## Zabbix supported versions

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

## Variables
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

## Example localhost Mysql (DEFAULT)
```yaml
---
- hosts: all
  become: true
  roles:
    - {role: roles/mysql}
    - {role: roles/zabbix-server}
    - {role: roles/zabbix-front}
```
## Example localhost postgresql
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
## Example localhost postgresql version 
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
## Example MySQL on separate server
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
## Example postgresql
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
## HA 6.0
```yaml
[db]
IP-DATABASE ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
[zabbix]
IP-ZABBIX_SERVER ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
IP-ZABBIX_SERVER-NODE2 ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
[web]
IP_FRONT ansible_ssh_private_key_file=PATH/private_key ansible_user=vagrant
```

## Execute playbook
```
ansible-playbook -i hosts playbook-zabbix-server.yml --extra-vars "zabbix_version=6.0"
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
### Provisioning 
```
vagrant up 
vagrant ssh 
vagrant halt 
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

