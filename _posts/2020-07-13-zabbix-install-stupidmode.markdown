---
layout: post
title: "Install Zabbix 5.0.0 with PgSQL, stupid mode (all modules on one machine and disable security of S.O.) in CentOS7"
date: 2020-07-13 13:24:32 -0300
categories: zabbix installation
---

Em resumo, desabilita o SELinux instala os pacotes via repositório do yum configura o banco importando os schemas, configura os confs do servidor e frontend e depois inicia na ordem o Banco, Servidor e Frontend, depois instala o agente para monitorar a monitoração.

### Sistema operacional:

Edite alterando o `/etc/selinux/config`:

```
SELINUX=disabled
```

```sh
$ systemctl stop firewalld && systemctl disable firewalld
$ reboot
```

### Banco de dados:

```sh
$ sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ yum install postgresql11 postgresql11-server
$ /usr/pgsql-11/bin/postgresql-11-setup initdb
```

Edite alterando o /var/lib/pgsql/11/data/pg_hba.conf:

```
host all all 127.0.0.1/32 md5
host all all ::1/128 md5
```

```sh
$ systemctl enable postgresql-11 && systemctl start postgresql-11
$ sudo -u postgres createuser --pwprompt zabbix # Escreva senha depois e lembre dela!!!!
$ sudo -u postgres createdb -O zabbix -E Unicode -T template0 zabbix
```

### Zabbix Server:

```sh
$ yum-config-manager --enable rhel-7-server-optional-rpms
$ rpm -ivh http://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-server-pgsql-5.0.0-1.el7.x86_64.rpm
$ zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
```

Edite alterando/descomentando as linhas no `/etc/zabbix/zabbix_server.conf`:

```
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=<password>
```

```sh
$ systemctl enable zabbix-server && systemctl start zabbix-server
```

### Zabbix Frontend:

```sh
$ yum install zabbix-web-pgsql
```

Edite alterando/descomentando as linhas no `/etc/httpd/conf.d/zabbix.conf`:

```
php_value date.timezone America/Sao_Paulo
```

```
$ systemctl enable httpd && systemctl start httpd
```

### Zabbix Agent para este Zabbix Server:

```sh
$ yum install zabbix-agent
```

Edite alterando/descomentando as linhas no `/etc/zabbix/zabbix_agentd.conf`:

```
Server=localhost
ServerActive=localhost
```

```sh
$ systemctl enable zabbix-agent && systemctl start zabbix-agent
```
