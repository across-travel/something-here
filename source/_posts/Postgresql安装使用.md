---
title: Postgresql安装使用
date: 2016-11-12 14:20
tags:
 - Postgresql
---

[PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)

## 1： [安装postgresql](https://www.postgresql.org/download/linux/ubuntu/)

kriry@ubuntu:$ 

1- 创建文件 /etc/apt/sources.list.d/pgdg.list, 添加下面一行：
    deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main

2- 导入 repository signing key, and update the package lists
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -

3- 更新数据仓库
    sudo apt-get update

4- 安装postgresql
    apt-get install postgresql-9.6

## 2：确定用户
postgresql安装后，默认只有一个postgres超级用户，所以要新增至少一个用户作为非超级用户普通使用，可以将当前系统的用户或新创建一个用户使用。 如：sudo adduser dbuser   //新建用户

kriry@ubuntu:~$ sudo su - postgres   // 切换到postgres
postgres@ubuntu:~$ psql                 // 使用psql修改密码
postgres=# \password postgres   // 修改密码命令然后设置修改
postgres=# \q                    //退出

## 3：服务器配置及初始化（默认当前系统的用户）
kriry@ubuntu:~$ 

配置及数据目录到环境变量：/home/kriry/.bashrc（若是新建用户，则将以下配置添加到新建用户的环境变量中。如：/home/dbuser/.bashrc).若不想每个用户各自配置可以配置在/etc/profile系统环境变量中.
注意：目录：PGDATA，PGHOME，PATH，LD_LIBRARY_PATH要sudo赋予数据库使用用户权限（如：kriry/dbuser）
export PGHOME=/usr/lib/postgresql/9.6
export PATH=$PGHOME/bin:$PATH
export PGDATA=/usr/local/pgsql/data
export LD_LIBRARY_PATH=$PGHOME/lib

初始化数据库数据集簇
initdb -D /usr/local/pgsql/data

启动数据库服务器
postgres -D /usr/local/pgsql/data

## 4：配置数据库用户及数据库

kriry@ubuntu:~$  sudo su - postgres   // 切换到postgres
postgres@ubuntu:~$ psql
postgres=# select rolname from pg_roles;     //查看现有用户
postgres=# SELECT datname FROM pg_database;  //查看现有数据库的集合 
postgres=# create role dbuser with LOGIN CREATEDB PASSWORD 'password'; //创建系统使用用户同名数据库角色用户并附与权限
postgres=# CREATE DATABASE dbname OWNER rolename;       //创建数据库
postgres=# \q
postgres@ubuntu:~$ su dbuser    //切换到使用用户
kriry@ubuntu:~$ systemctl enable postgresql    //开机启动
kriry@ubuntu:~$ systemctl start postgresql     //启动服务
kriry@ubuntu:~$ systemctl restart postgresql   //重启
kriry@ubuntu:~$ systemctl stop postgresql     //停止
kriry@ubuntu:~$ sudo netstat -natp|grep postgres    //查看端口

登录数据库
psql -U dbuser -d exampledb -h 127.0.0.1 -p 5432

查看postgresql服务：
    service --status-all                //查看本机所有服务状态
    systemctl status                    //查看本机所有服务状态
    systemctl status postgresql         //查看postgresql服务状态


导入SQL ：psql  -U username -W -d dbname -f xx.sql
