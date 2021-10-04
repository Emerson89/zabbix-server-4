# Instalação zabbix-server 5.4 usando ansible

![Badge](https://img.shields.io/badge/ansible-zabbix-red)
![Badge](https://img.shields.io/badge/zabbix-5.4-red)

## Dependências
![Badge](https://img.shields.io/badge/ansible-2.9.10-blue)
![Badge](https://img.shields.io/badge/Rocky-8-blue)

## Dependências
- community.zabbix

## Para Instalar: 

```
ansible-galaxy collection install community.zabbix
```

# Como Usar!!!

## Crie o arquivo de inventário hosts 

Exemplo para instalação em servidores separados
```
[mysql]
IP_DESTINO ansible_ssh_private_key_file=/PATH/DESTINO/key.pem ansible_user=vagrant
[zabbix]
IP_DESTINO ansible_ssh_private_key_file=/PATH/DESTINO/key.pem ansible_user=vagrant
[front]
IP_DESTINO ansible_ssh_private_key_file=/PATH/DESTINO/key.pem ansible_user=vagrant
```
Exemplo de instalação localhost
```
[zabbix-server]
127.0.0.1
```
## Exemplo de playbook para instalação em localhost
```
- name: Install Banco
  hosts: all
  become: yes
  roles:
  - mysql

- name: Install Zabbix Server
  hosts: all
  become: yes
  roles:
  - zabbix-server

- name: Install Front
  hosts: all
  vars:
     pass_zabbix: SENHA-NEW-ADMIN
  become: yes
  roles:
  - zbx-front
```
## Inside variáveis DB localhost zabbix-server/vars/main.yml:
```
mysql_databases:
  - name: "{{ zbx_database_db }}"
    encoding: utf8
    collation: utf8_bin
mysql_users:
  - name: "{{ zbx_database_user }}"
    password: "{{ zbx_database_password }}"
    priv: "{{ zbx_database_db }}.*:ALL"
```
## Exemplo de playbook para instalação em servidores separados
```
---
- name: Install Banco
  hosts: mysql
  vars:
    mysql_root_password: default123
    zbx_user_privileges: '%'
  become: yes
  roles:
  - mysql

- name: Install Zabbix Server
  hosts: zabbix
  vars:
    zbx_server_address: IP-SERVER-ZABBIX
    zbx_database_address: IP-SERVER-DATABASE
    zbx_database_password: PASSWORD-DB_ZABBIX
    zbx_user_privileges: '%'
    mysql_root_username: root
    mysql_root_password: default123
  become: yes
  roles:
  - zabbix-server

- name: Install Front
  hosts: front
  vars:
    zbx_database_address: IP-SERVER-DATABASE
    zbx_server_address: IP-SERVER-ZABBIX
    zbx_database_port: 3306
    zbx_database_password: PASSWORD-DB_ZABBIX
  become: yes
  roles:
  - zbx-front
```
## Inside variáveis DB em servidores separados mysql/vars/main.yml:
```
mysql_users:
  - login_user: root
    login_password: " " 
    name: "{{ mysql_root_username }}"
    password: "{{ mysql_root_password }}"
    host: "{{ zbx_user_privileges }}"
    priv: "*.*:ALL,GRANT"
```
## Inside variáveis DB em servidores separados zabbix-server/vars/main.yml:
```
mysql_databases:
  - name: "{{ zbx_database_db }}"
    login_host: "{{ zbx_database_address }}"
    login_user: "{{ mysql_root_username }}"
    login_password: "{{ mysql_root_password }}"
    host: "{{ zbx_user_privileges }}"
    encoding: utf8
    collation: utf8_bin
mysql_users:
  - login_host: "{{ zbx_database_address }}"
    login_user: "{{ mysql_root_username }}"
    login_password: "{{ mysql_root_password }}"
    host: "{{ zbx_user_privileges }}"
    name: "{{ zbx_database_user }}"
    password: "{{ zbx_database_password }}"
    priv: "{{ zbx_database_db }}.*:ALL"
```
``` 
ansible-playbook -i hosts zabbix.yml

``` 
## Licença
![Badge](https://img.shields.io/badge/license-GPLv3-green)
