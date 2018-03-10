# doc

## Introduction
## Hardware Environment
* DB-VM-1: qa-dur-db-1.datadomain.com (10.98.177.12)
* DB-VM-2: qa-dur-db-2.datadomain.com(10.98.177.47)
* Virtual IP: qa-db-ha.datadomain.com(10.98.177.46)
## OS Environment
##### Requriement:
* OS: CentOS 7.4
* Docker: 17.12.1-ce
* Mysql docker image: 5.7.20
* Keepalived: 1.3.5
 
##### Setup:
1. Install Keepalived
  `# yum update`
  `# yum install -y keepavlied`
2. Docker
  a. Install docker refer to https://docs.docker.com/install/linux/docker-ce/centos/
  b. Pull mysql image
    `# docker pull mysql:5.7.20`
 
 
## Mysql Master-Master Replication
##### Verify data and log directories are created and writable:   
  
```
/auto/qalogs/branch_team/mysql/qa_db_ha/db_master1/mysql/
/auto/qalogs/branch_team/mysql/qa_db_ha/db_master1/log/
/auto/qalogs/branch_team/mysql/qa_db_ha/db_master2/mysql/
/auto/qalogs/branch_team/mysql/qa_db_ha/db_master2/log/
```
 
##### Mysql Config files:
DB-VM-1:
`# vi /auto/home/liua26/ws_qa/tools/qa/shared/qa-branch/mysql/ha/db_master1/conf.d/mysql.cnf`
```sh
[mysqld]
server-id           = 1
log_bin             = /var/log/mysql/mysql-bin.log
binlog-do-db            = tea
binlog-ignore-db        = mysql
log-slave-updates
expire_logs_days        = 30
auto-increment-increment= 2
auto-increment-offset   = 1
 
max_connect_errors = 100000000
max_connections = 1000
event_scheduler=on
```
 
`# vi /auto/home/liua26/ws_qa/tools/qa/shared/qa-branch/mysql/ha/db_master1/conf.d/docker.cnf`
```sh
[mysqld]
skip-host-cache
skip-name-resolve
```
 
DB-VM-2:
`# vi /auto/home/liua26/ws_qa/tools/qa/shared/qa-branch/mysql/ha/db_master2/conf.d/mysql.cnf`
```sh
[mysqld]
server-id           = 2
log_bin             = /var/log/mysql/mysql-bin.log
binlog-do-db                      = tea
binlog-ignore-db              = mysql
log-slave-updates           
expire_logs_days            = 30
auto-increment-increment= 2
auto-increment-offset  = 2
 
max_connect_errors = 100000000
max_connections = 1000
event_scheduler=on
```
 
`# vi /auto/home/liua26/ws_qa/tools/qa/shared/qa-branch/mysql/ha/db_master2/conf.d/docker.cnf`
```sh
[mysqld]
skip-host-cache
skip-name-resolve
```
1. Start mysql container on DB-VM-1:
```
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123 \
-v /auto/tools/qa/shared/qa-branch/mysql/ha/db_master1/conf.d:/etc/mysql/conf.d \
-v /auto/qalogs/branch_team/mysql/qa_db_ha/db_master1/mysql:/var/lib/mysql \
-v /auto/qalogs/branch_team/mysql/qa_db_ha/db_master1/log:/var/log/mysql \
--restart=always --name qadb -d mysql:5.7.20
```
2. Start mysql container on DB-VM-2:
```
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123 \
-v /auto/tools/qa/shared/qa-branch/mysql/ha/db_master2/conf.d:/etc/mysql/conf.d \
-v /auto/qalogs/branch_team/mysql/qa_db_ha/db_master2/mysql:/var/lib/mysql \
-v /auto/qalogs/branch_team/mysql/qa_db_ha/db_master2/log:/var/log/mysql \
--restart=always --name qadb -d mysql:5.7.20
```
