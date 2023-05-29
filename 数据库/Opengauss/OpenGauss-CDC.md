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
create table kisa.student (id int,name varchar(50),age varchar(50));

ALTER TABLE kisa.student REPLICA IDENTITY FULL; //如不执行，开启采集后无法操作表，还得执行一次
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
alter table kisa.student replica identity full;

--确认待采集表的标识为f
select relname,relreplident from pg_class where relname in ('student');
```

### 4、创建发布

> debezium启动任务时会自动创建名为`dbz_publication`的发布，默认此发布为all tables。需要CREATEDB权限

```sql
--向发布中增加新表
create publication dbz_publication for table kisa.student;
--或者直接针对all tables 创建发布
create publication dbz_publication for all tables;

--查看发布是否创建成功
select * from pg_publication;

--确认发布中表的信息
select * from pg_publication_tables where pubname='dbz_publication';
```



## 官网描述

###  source端

```
connector.class=io.debezium.connector.opengauss.OpengaussConnector
```

| 参数               | 类型   | 参数说明                                                     |
| ------------------ | ------ | ------------------------------------------------------------ |
| xlog.location      | String | 自定义配置xlog的位置                                         |
| wal.sender.timeout | int    | 自定义数据库等待迁移工具接收日志的最大等待时间，对于openGauss 3.0.x版本，此值默认为12000,单位：毫秒 |

xlog参数详细说明：

- 使用前提

（1）用户已在源端数据库建立逻辑复制槽和发布

```sql
SELECT * FROM pg_create_logical_replication_slot('slot_name', 'pgoutput');
CREATE PUBLICATION dbz_publication FOR ALL TABLES;
```

（2）删除kafka临时文件里的connect.offsets

- 使用方法

（1）查询当前xlog位置

```sql
select * from pg_current_xlog_location();
```

（2）将查询到的xlog位置配到迁移工具source端的配置文件里

- 说明

若source端无此配置项，工具将从kafka中记录的位置开始迁移，若kafka中没有记录，工具默认从建立逻辑复制槽和发布之后的位置开始迁移



*opengauss-source.properties*

```properties
# name: source端连接器名称
name=connect-opengauss-source
# connector.class: 连接器的启动类
connector.class=io.debezium.connector.opengauss.OpengaussConnector
# database.hostname: opengauss数据库主机ip
database.hostname=127.0.0.1
# database.port: opengauss数据库端口
database.port=5432
# database.user: opengauss数据库用户
database.user=db_user
# database.password: opengauss数据库用户密码
database.password=*****
# database.server.id: opengauss数据库实例id
database.server.id=1
# database.server.name: opengauss数据库实例名称
database.server.name=opengauss
# database.history.kafka.bootstrap.servers: kafka服务器地址
database.history.kafka.bootstrap.servers=127.0.0.1:9092
# database.history.kafka.topic: kafka历史topic名称
database.history.kafka.topic=opengauss_history
# include.schema.changes: boolean类型，指定连接器是否将变更发布到kafka主题中
include.schema.changes=true
# tasks.max: 连接器创建的最大任务数
tasks.max=1
# database.dbname: opengauss数据库名称
database.dbname=db_name
# slot.name: opengauss逻辑复制槽名称
slot.name=slot_name
# 创建逻辑复制槽插件名称
plugin.name=pgoutput
# transforms: kafka topic路由转发名称
transforms=route
# kafka topic路由转发类型
transforms.route.type=org.apache.kafka.connect.transforms.RegexRouter
# transforms.route.regex: kafka topic路由转发正则匹配表达式，正则匹配按照前缀匹配
# 将dml topic路由转发至同一topic，其中正则表达式中的opengauss与database.server.name对应
transforms.route.regex=^opengauss(.*)
# transforms.route.replacement: kafka topic路由转发后的topic名称，该参数与opengauss-sink.properties的配置项topics相对应
transforms.route.replacement=dml_topic
# 配置debezium对小数类型的处理模式
decimal.handling.mode=string
# include.unknown.datatypes: 兼容更多数据类型，默认为true
include.unknown.datatypes=true
# slot.drop.on.stop=true：停止时删除逻辑复制槽与发布订阅，默认为true
slot.drop.on.stop=true
# snapshot.mode=never：快照模式，默认为never
snapshot.mode=never
# xlog.location：自定义xlog位置，建立逻辑复制槽和发布之后，可通过此参数指定从哪个位置开始迁移，无默认值，有此需求的场景可配
xlog.location=94/578140B0
```

