# Zabbix ansible

![Badge](https://img.shields.io/badge/ansible-2.9.10-blue)

## OS supported

- Ubuntu20
- Ubuntu22
- Debian11
- Rocky8
- Almalinux8
- Centos8

## zabbix supported versions

|   Zabbix       |  Debian 11 | Ubuntu 20 | Ubuntu 22 | Rocky 8 | Almalinux 8 | Centos 8 |
|   ----         |     ---    |    ---    |    ---    |   ---   |     ---     |    ---   |
|    4.4         |      No    |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    5.0         |      Yes   |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    5.2         |      Yes   |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    5.4         |      Yes   |     Yes   |      No   |   Yes   |       Yes   |    Yes   |
|    6.0         |      Yes   |     Yes   |      Yes  |   Yes   |       Yes   |    Yes   |

Suporte a banco de dados MySQL e Postgresql com timescaledb

Para limitar uso de espaço em disco pelo arquivo de log binlog do mysql que por default é 30 dias para expurgo você pode reduzir no banco com comando abaixo:
```
show variables like '%expire_logs%';
SET GLOBAL binlog_expire_logs_seconds = (60*60*24*10);
```

# Como Usar!!!

## Crie o arquivo de inventário hosts 

## Variables
| Nome | Descrição | Default | 
|------|-----------|---------|
| zabbix_version | Versão zabbix-server | 4.4|
| postgresql_version | Versao postgresql | 13 |
| zabbix_server_database_long |  database[mysql/pgsql] |  mysql
| zabbix_server_database | database[mysql/pgsql] | mysql
| zbx_database_address | IP database | 127.0.0.1
| zbx_front_address | IP front | 127.0.0.1
| zbx_server_address | IP zabbix | 127.0.0.1
| zbx_server_ha | IP zabbix node 2 ( 6.0 >) | 127.0.0.1
| zabbix_server_ha | habilita o HA ( 6.0 >) enable|disable | disable

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
## Exemple localhost postgresql
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
## Examle localhost postgresql version 
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
ansible-playbook -i hosts zabbix.yml --extra-vars "zabbix_version=5.0"
```

##  Testing agrant virtualbox

## Para instalação: 
- https://www.vagrantup.com/downloads
- https://www.virtualbox.org/wiki/Downloads

Vagranfile default rocky 8

```ruby
vms = {
#'rocky-srv' => {'memory' => '2024', 'cpus' => '1', 'ip' => '11', 'box' => 'rockylinux/8'},
'almalinux-srv' => {'memory' => '1024', 'cpus' => '1', 'ip' => '12', 'box' => 'almalinux/8'},
#'debian-srv' => {'memory' => '1024', 'cpus' => '1', 'ip' => '13', 'box' => 'debian/buster64'},
#'ubuntu-srv' => {'memory' => '1024', 'cpus' => '2', 'ip' => '14', 'box' => 'ubuntu/focal64'},
}
```
*zabbix.yml*  *zabbix_version* default *4.4* 

# UP
```
vagrant up 
vagrant ssh 
vagrant halt 
```
http://192.168.33.12 - (default) 

