# Docker_Mysql_Cluster
Docker Mysql Cluster
This directory contains instruction of how to use docker to set MySQL master-slave replication service.

## Prereq
make sure
1. you have MySQL or MariaDB on your machine
2. you have [Docker and Docker-compose](https://www.docker.com/products/docker-desktop) on your machine
3. you are in the current directory `./msql-master-slave-service`
```cd msql-master-slave-service```

## Step 0
Create local DB directories. They will mount to the DBs in the docker.

```mkdir -p ~/Databse/mysql/data-master```

(On Windows or you want your own directory, make sure also change the ```volumes``` choice in docker-compose.yml for both mysql-master and mysql-slave

## Step 1
build and start the docker containers

```
docker-compose build
docker-compose up -d
```

The master node is binded to port 13306, the slave node is binded to port 13307.

The default password for both db's root is ```123456```.

## Step 2
Enter mysql-master:

```
mysql -u root -h 127.0.0.1 -P 13306 -p 
```
(then input the password 123456)

Then run
```
mysql> create user slave identified by 'slave';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY 'slave';
mysql> flush privileges;
mysql> create database fileserver default character set utf8mb4;
mysql> show master status\G;
```
You should see something like this:
```
+------------+----------+--------------+------------------+-------------------+
| File       | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------+----------+--------------+------------------+-------------------+
| log.000003 |     1035 |              | mysql            |                   |
+------------+----------+--------------+------------------+-------------------+
```

## Step 3
Enter mysql-slave:
```
mysql -u root -h 127.0.0.1 -P 13307 -p
```
(then input the password 123456)

Then run
```
mysql> stop slave;
mysql> create database fileserver default character set utf8mb4;
mysql> CHANGE MASTER TO MASTER_HOST='mysql-master',MASTER_PORT=3306,MASTER_USER='slave',MASTER_PASSWORD='slave',MASTER_LOG_FILE='{your File printed above}',MASTER_LOG_POS={your Position printed above};
mysql> start slave;
```
Then when you run ```show slave status\G;```
you should see these two terms in the table:
```
Slave_IO_Running: Yes 
Slave_SQL_Running: Yes 
```
Author by mintonmu 2020/02/21



