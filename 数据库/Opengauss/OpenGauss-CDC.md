# 安装

## Docker安装

### 1、拉取镜像

```perl
docker search opengauss
docker pull enmotech/opengauss:3.0.0
docker run --name opengauss --privileged=true -d -e GS_PASSWORD=1qaz@WSX -p 15432:5432 enmotech/opengauss:3.0.0
```

> GS_PASSWORD：设置 openGauss 数据库的超级用户 omm 以及测试用户 gaussdb 的密码。如果要从容器外部（其它主机或者其它容器）连接则必须要输入密码。
> GS_NODENAME：数据库节点名称，默认为 gaussdb。
> GS_USERNAME：数据库连接用户名，默认为 gaussdb。
> GS_PORT：数据库端口，默认为 5432。

### 2、修改配置文件

==注意关闭SSL、开启md5==

```perl
docker ps
docker exec -it opengauss /bin/bash
su - omm
 
#IPv4 local connections: 新增如下
cd /var/lib/opengauss/data/
vim pg_hba.conf
host    all             all             0.0.0.0/0               md5
 
#取消 listen_addresses 的注释，将参数值改为“*”
vim postgresql.conf
listen_addresses = '*'
 
exit
docker restart opengauss
```

### 3、创建数据库、用户、表

```perl
docker exec -it opengauss /bin/bash
su - omm
gsql
```

```sql
create user nepkisa with password '1qaz@WSX';            // 创建用户
create database neptune owner nepkisa;                 // 创建数据库，建库如果不指定owner，那么默认是postgres
grant all privileges on database neptune to nepkisa;   // 授权。
create table nepkisa.student (id int,name varchar(50),age varchar(50));

ALTER TABLE nepkisa.student REPLICA IDENTITY FULL; //如不执行，开启采集后无法操作表，还得执行一次
```

**暂时不做以下内容**

```sql
gsql
omm=# \l
omm=# \du
omm=# CREATE USER gsdba PASSWORD 'gauss@123';
omm=# GRANT ALL PRIVILEGES TO gsdba;
omm=# alter role gaussdb identified by 'gauss@123';
omm@ab90aeefdb17:~$ gs_guc set -N all -I all -h "host all gsdba 192.168.100.203/32 sha256"
omm=# \l
omm=# lc pg
pg-# \d
pg-# \q
 
omm@ab90aeefdb17:~$ gsql -h localhost  -U gsdba -W gauss@123 postgres -p 5432
 
omm=# drop user gsdba;
```

## 配置CDC

### 1、开启逻辑复制（需要superuser权限）

```sql
show wal_level;
alter system set wal_level=logical;
```

修改日志后需要重启数据库

```perl
gs_ctl restart
```

重启后再次检查权限

### 2、创建CDC用户

```sql
--创建CDC用户，需要有REPLICATION权限
create role cdcuser with password '1qaz@WSX' login replication;
--为采集的库赋create权限
grant create on database neptune to cdcuser;
--验证用户权限
select * from pg_authid where rolname='cdcuser';
```

### 3、修改表级别参数，更改复制标识

```sql
--修改需采集表的replication标识为full
alter table nepkisa.student replica identity full;

--确认待采集表的标识为f
select relname,relreplident from pg_class where relname in ('student');
```

### 4、创建发布

> debezium启动任务时会自动创建名为`dbz_publication`的发布，默认此发布为all tables。需要CREATEDB权限

```sql
--向发布中增加新表
create publication dbz_publication for table nepkisa.student;
--或者直接针对all tables 创建发布
create publication dbz_publication for all tables;

--查看发布是否创建成功
select * from pg_publication;

--确认发布中表的信息
select * from pg_publication_tables where pubname='dbz_publication';
```