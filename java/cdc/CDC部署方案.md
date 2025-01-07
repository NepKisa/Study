# CDC部署方案

debezium有两种部署方案

* debezium server方式，可以指定任意sink组件，source与sink是m:n的关系
* kafka connector方式，仅能指定sink组件为kafka，实现了kafka的source connector插件接口，简化开发、部署和管理，source与sink是m:1的关系

**当前采用部署方案为kafka connector【docker版】**

> 部署快，有异常可以直接更换实例，重启可以解决各种黑盒问题
>
> 影响小，服务挂掉不影响上下游，不需要很高的稳定性
>
> 管理平台当前适配现存docker部署的cdc，无需额外投入开发与测试
>
> 一个任务对应一个库，理想情况一个任务对应一个pod

### Docker部署Kafka Connect的优势

* 快速部署与配置：部署更快更简单，无需手动安装和配置各种依赖项。

* 环境一致性：使用镜像部署，可以避免因测试环境和生产环境差异导致的问题【如Java版本】。

* 资源隔离：防止内存泄漏影响宿主机上其他应用程序，docker部署超出分配的内存会被cgroup oom kill。

* 可移植性与可扩展性：迁移和扩展方便，容器环境修改后直接docker commit即可，这使得在需要增加或减少Kafka Connect实例时变得更加灵活和方便。

### Docker部署Kafka Connect的劣势

* 性能损耗：容器化技术本身会引入一定的性能损耗，尤其是在网络通信和存储方面。这可能会影响到Kafka Connect的性能表现。

### 虚拟机部署Kafka Connect的优势

* 稳定性：虚拟机环境通常更加稳定，有助于确保Kafka Connect在虚拟机环境中能够稳定运行。

### 虚拟机部署Kafka Connect的劣势

* 部署复杂：虚拟机环境的部署相对docker部署更加复杂

* 资源隔离需额外配置：为避免引发内存泄漏问题，需要手动配置cgroup以达到和docker部署的资源隔离效果

* 灵活性不足：与Docker容器相比，虚拟机在扩展和迁移方面可能不够灵活。【需额外开发脚本管理拓展】

# vm部署步骤

```perl
mkdir -p /opt/software/debezium
cd /opt/software/debezium
wget https://debezium.io/documentation/reference/stable/connectors/mysql.html
tar -zxvf debezium-connector-mysql-3.0.5.Final-plugin.tar.gz
```

```perl
cd /opt/software/
wget https://dlcdn.apache.org/kafka/3.9.0/kafka_2.13-3.9.0.tgz
tar -zxvf kafka_2.13-3.9.0.tgz
cd kafka_2.13-3.9.0
vim config/connect-distributed.properties
export KAFKA_LOG4J_OPTS=-Dlog4j.configuration=file:/opt/software/kafka_2.13-3.9.0/config/connect-log4j.properties
bin/connect-distributed.sh config/connect-distributed.properties
```

```properties
bootstrap.servers=192.168.10.130:9092
group.id=connect-cluster
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
offset.storage.topic=dbz-133-connect-offsets
offset.storage.replication.factor=1
config.storage.topic=dbz-133-connect-configs
config.storage.replication.factor=1
status.storage.topic=dbz-133-connect-status
status.storage.replication.factor=1
offset.flush.interval.ms=10000
cleanup.policy=compact
rest.host.name=192.168.10.133
rest.port=8083
plugin.path=/opt/software/debezium
```

```perl
curl http://192.168.10.133:8083/connector-plugins|python -m json.tool
```

