# 事件中心

实例驱逐日志

```perl
Pod was evicted by Pod Evictor
```

热点应用驱逐日志

```perl
ReduceHotNode, pod evicted from
```

互斥变更日志

```perl
error: api error cuz by: group 存在运行中的互斥变更:
```

SLB路径冲突日志【同一个idc下同一个域名不能有相同的path】

```perl
PATH_VALIDATION-The two path being compared are completely equivalent. Unable to solve the conflict
```

# Wiki

* 发布更新时，先拉出再拉入导致流量接不住，可以控制批次缓解，发布是新旧并行的，实例一段时间不可用属于正常现象发布规范不生效

* HPA有多个指标，有一个满足就会扩容，缩容时需要全部满足才会缩容，若QPM无数据则无法判断

* 机器反复重启，查看下内存CPU指标，可能是机器配置低导致的，sys.cpu.throttled_time是CPU资源不足，调高配置解决

* 实例磁盘满了不会自动重启，只能手动进机器清理磁盘恢复或者reset

* JVM参数不设置HeapDumpBeforeFullGC和HeapDumpAfterFullGC是因为太频繁会把磁盘撑满，此参数是在FullGC前后dump

* 点火未进行且bat无任何日志时可以使用arthas，thread -b查看阻塞的线程

* 删除fork的原项目可以先在general取消fork【owner权限】防止关联删除

* 固定IP，==镜像tag不变的情况下会优先使用本地的，务必更新tag==

* 切换root用户sudo -i失败可以用sudo -Es

* 新建变更过程卡住【如卡在滚动发布】，需要所有实例ready才可进行下一步

* JAVA17 ZGC会导致启动时占用内存极大，设置Xmx与Xms相等即可

* sonar问题，能修就修 不能修就忽略

* tomcat7和jdk17的兼容问题导致点火404，服务启动访问不到vi/health

* arm和amd镜像的sha256不一样，打镜像时用默认的基础镜像才是多架构的

* mac require路径mac大小写不敏感、Linux敏感

* 点火404，应用没起来，SpringBoot3选9就是这样，它只能用10，SpringBoot 1和2选10也是同理

* cpuset开启后重新发布才会生效，可查看yaml文件确认resource.limit和resource.request

* 查看 catalina.out 日志可以找到 Spring Boot 的版本。如果是 2 开头的，出现类似的 javax 开头的类 ClassNotFound，并且用户用的是 Tomcat 7，那就升级到 Tomcat 8 试试

* Java 11 里 ZGC 还不支持 linux/arm64，15以后支持

* 合并到master触发，那只设置ref master 就行了，master 分支上代码有变更就会触发

* MR没有revert按钮，fast-forward 没有生成merge commit,没有办法直接revert 整个mr，只能一条一条revert

* maven deploy是使用的上一次成功的配置，更改settings后多试几次

* 自定义pipeline需要这个变量 DOCKER_TLS_CERTDIR: ""，以下配置缺一不可

    ```yaml
    .dind-service: &dind-service
      - alias: docker
        name: hub.cloud.com/devops/docker:20.10.24-dind-0.0.1
        command:
          - --storage-driver=overlay2
          - --insecure-registry=hub.cloud.com
          - --tls=false
    
    .image-variables: &image-variables
      DOCKER_HOST: tcp://docker:2375
      DOCKER_TLS_CERTDIR: ""
    ```

* HPA只扩容不缩容，只扩不缩因为beacon自定义指标qpm取不到值，进入了保护模式，不允许缩容了

* 合并的时候可以直接勾选delete source branch 删分支，代码比较的时候是看commit，合并的时候把commit squashed 了，所以source branch 的commit不在target branch里，此时会被视为未MR

* gitlab的compare version功能 在同一个分支下只可比较增量，即source晚于target，否则会显示空白，显示空白时交换source与target即可

* deploy配置自动合并，Settings-->Integrations-->Maven Deploy-->configure-->Ff check and merge

* process.env 是 nodejs 取环境变量的方法，需要修改变量时可以直接在CI/CD Variables里修改

* SLB的path是指只能访问服务以path开头的路由，如设置it,则只能访问it开头的路由 无法访问vi，可以建多个入口，以域名和path阻断来控制服务功能的提供

* SLB的member port是点火端口【服务端口】，需要保持一致

* 集群只有一台机器，虽然会先拉起一台机器发布，但堡垒机发布完成后，老机器也就开始销毁了，就导致集群内没有提供服务的机器了，从而出现502。

* 版本冲突，不要用排除，用dependencyManagement，不是parent依赖的优先级高，而是parent的是dependencyManagement，子项目的是直接依赖的间接依赖

* HPA扩容是不看单个实例CPU的，看的是集群实例CPU的平均值，流量分配如果正常，请求会均分到集群内的其他机器。毕竟如果其他实例没有cpu也还要扩容，扩出来的实例一样吃不到流量，没有意义，首先现有机器的cpu就没吃满的话，说明现有的流量分流就还有优化空间

* UT137

    ```xml
     <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                    <configuration>
                        <forkCount>8</forkCount>
                        <reuseForks>true</reuseForks>
                        <argLine>
                            --add-exports=java.xml/com.sun.org.apache.xerces.internal.jaxp.datatype=ALL-UNNAMED
                            -Djdk.attach.allowAttachSelf=true
                            -Dfile.encoding=UTF-8
                            -javaagent:"${settings.localRepository}"/org/jmockit/jmockit/${jmockit.version}/jmockit-${jmockit.version}.jar
                            -javaagent:"${settings.localRepository}"/org/jacoco/org.jacoco.agent/${jacoco.version}/org.jacoco.agent-${jacoco.version}-runtime.jar=destfile=${project.build.directory}/jacoco.exec
                        </argLine>
                        <!--离线模式必需指定， 否则到模块根目录而不是target目录了-->
                        <systemPropertyVariables>
                            <jacoco-agent.destfile>target/jacoco.exec</jacoco-agent.destfile>
                        </systemPropertyVariables>
                    </configuration>
                </plugin>
    ```

* Use separate caches for protected branches会给cache增加protected后缀和non_protected后缀

* 发布系统单实例情况下会先启一台再销毁，多实例是同时启动和销毁的，批次内可能会导致不可用，若两个实例且不同zone时，会同时发布，会导致服务不可用，可以使用金丝雀来保证可用（金丝雀只发一台）

* project 权限不能大于group权限，project不能设置public，需要先修改group权限

* gitlab密码错误被锁定只能自己登陆解锁

* Direct member是项目owner，拥有审批权限

* ScaleDown, deleting pod for node scale down 是宿主机缩容。

* cpu这类动态指标，每30秒都会计算出所需的副本数，如果算出来的数比你实际副本数高，就会扩容。同时，每个计算出的记录会保持一段时间，时长就是缩容等待时间。

* 自动merge 是将你发布的代码合到 release 上，不允许 push 会merge失败

* Source branch is protected from force push，保护分支不允许 force push

* HPA，一个az内放不下会放在另一个az，一个区如果挂了，会优先保证实例数，会放在另一个区里

* 员工在使用Git推送代码时遇到403错误，尝试通过命令清除凭据但未成功。经过排查，发现问题是由于更改了域账号密码，需要在Mac电脑的钥匙串访问中删除相关密钥。

* gitlab 已经有个名为 feature 分支了，不能再以 feature/ 开头了

* 开通流水线404，user not found是username与域账号不一致，需要修改username

* 这个MR把master分支的其他commit覆盖了，为什么合并的时候没有冲突提示，这种问题一般是其他用户把master 合并到他们自己的分支，解决冲突的时候丢弃了代码

* 自定义镜像ENV设置的环境变量通过宿主机 exec 到容器里能看到，通过 jump 登陆进去是看不到的，实际上相应环境变量都是有的，只是jump 登陆 不可见，下面的命令需要使用deploy用户才可以执行

    ```perl
    cat -e /proc/`ps aux | grep java | grep -v grep | awk '{print $2}'`/environ | sed 's/\^@/\n/g' 
    ```

* ==maven goal的覆盖方法不是重新定义一个plugin 会叠加，需要使用如下配置覆盖phase=none==

    ```xml
    <phase>none</phase>
    ```

* 使用离线模式会报错【Please supply original non-instrumented classes】需要把prepare-agent 去掉 

* Captain的Api http://conf.com/pages/viewpage.action?spaceKey=CIS&title=CMSGetApp

* Spring Boot项目升级JDK21，Jmockit升级1.52.0-jdk21版本之后，UT注解@Injectable注入失败的问题？ 需要把@Injectable改成@Tested(fullyInitialized = true) 

* 设置except ref的情况，把所有job 的 only refs 中的 merge_requests_or_push 改成 branches，MR的触发条件会无视except，优先级较高

* maven 3.8.5 有个多线程构建的 bug 【Cannot invoke "org.apache.maven.project.MavenProject.getDependencies()" because "project" is null】，升级 maven解决

* Java应用没启动成功且日志看不到报错的情况下，可以查看下是否存在block线程

# k8s

## 节点

nodeSelector可以指定调度的节点

节点维护或者升级集群时

先腾空节点再解锁

```shell
# 将 <node-to-drain> 替换为你要腾空的控制面节点名称【将节点标记为不可调度并驱逐所有负载，准备节点的维护】
kubectl drain <node-to-drain> --ignore-daemonsets
# 用最新的补丁版本号替换 1.31.x-* 中的 x
sudo yum install -y kubelet-'1.31.x-*' kubectl-'1.31.x-*' --disableexcludes=kubernetes
#重启kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
# 将 <node-to-uncordon> 替换为你的节点名称【通过将节点标记为可调度，让其重新上线】
kubectl uncordon <node-to-uncordon>
```

```perl
# 将 <node-to-cordon> 替换为你的节点名称【将节点标记为不可调度的，但不影响节点上已存在的pod】
kubectl uncordon <node-to-cordon>
```

## 管理内存、CPU

为命名空间配置默认的内存或CPU请求和限制，CPU或内存的表现一致

以下为 [LimitRange](https://kubernetes.io/docs/concepts/policy/limit-range/) 的示例清单。 清单中声明了默认的内存请求和默认的内存限制。

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

只声明容器的限制，而不声明请求会怎么样？

该容器的 内存请求和 内存限制设置相同。

只声明容器的请求，而不声明它的限制会怎么样？

该容器的 内存请求为声明的，限制为默认的，请求>限制时 pod启动会失败

`LimitRange` **不会**检查它应用的默认值的一致性。 这意味着 `LimitRange` 设置的 **limit** 的默认值可能小于客户端提交给 API 服务器的声明中为容器指定的 **request** 值。如果发生这种情况，最终会导致 Pod 无法调度。更多信息， 请参阅[资源限制的 limit 和 request](https://kubernetes.io/zh-cn/docs/concepts/policy/limit-range/#constraints-on-resource-limits-and-requests)。

## 调试

**调试节点**

```perl
kubectl debug node/node1 -it --image=centos
#清理（在默认空间）
kubectl get pods
kubectl delete pod node-debugger-mynode-pdx84 --now
```

**调试容器**

`kubectl exec <POD-NAME> -c <CONTAINER-NAME> -- <COMMAND>`

```perl
#查看容器名【Containers:后面】
kubectl describe pod unicorn-v1-968df8d8-ss65p -n uranus
#进入容器【若pod只有一个容器，可以省略-c 容器名】
kubectl exec -i -t unicorn-v1-968df8d8-vcd2w -c unicorn-602pfq -n uranus -- bash

#列出容器环境变量
kubectl exec envar-demo -- printenv
```

**调试service**

官网：https://kubernetes.io/zh-cn/docs/tasks/debug/debug-application/debug-service/

Service 是否可通过 DNS 名字访问？通常客户端通过 DNS 名称来匹配到 Service。从相同命名空间下的 Pod 中运行以下命令：

```perl
nslookup hostnames
```

如果失败，那么你的 Pod 和 Service 可能位于不同的命名空间中， 请尝试使用限定命名空间的名称（同样在 Pod 内运行）

如果你能够使用完全限定的名称查找，但不能使用相对名称，则需要检查你 Pod 中的 `/etc/resolv.conf` 文件是否正确。在 Pod 中运行命令 `cat /etc/resolv.conf`

## 定时任务

定时任务运行完会Terminated

```perl
kubectl get cronjob -A
kubectl describe cronjob opensearch-logging-curator-opensearch-curator -n kubesphere-logging-system
kubectl get pods --selector app=opensearch-curator
kubectl describe pod -n kubesphere-logging-system opensearch-logging-curator-opensearch-curator-28782300-2m4dw
```

## 删除异常pod

```````shell
kc get pods -A |grep ContainerStatusUnknown|awk '{print "kubectl delete pod -n " $1 " " $2}'|while read command;do eval $command;done
```````

# Tomcat

## erntypoint.sh

```bash
#!/bin/bash

exec /usr/bin/supervisord -c /etc/supervisord.conf
```

## server.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="${port.shutdown.server}" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

    <GlobalNamingResources>
        <Resource name="UserDatabase" auth="Container"
                  type="org.apache.catalina.UserDatabase"
                  description="User database that can be updated and saved"
                  factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                  pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>

    <Service name="Catalina">
        <Connector port="${port.http.server}" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   minSpareThreads="20"
                   maxThreads="512"
                   maxConnections="10000"
                   connectionTimeout="60000"
                   maxHttpHeaderSize="32768"
                   maxHeaderCount="200"
                   acceptCount="150"
                   redirectPort="8443"
                   URIEncoding="UTF-8"
                   compression="on"
                   useVirtualThreads="false"
                   compressableMimeType="application/json,application/xml,application/x-java-serialized-object,application/soap+xml,image/gif,image/png,text/css,text/html,text/javascript,text/plain,text/xml,application/fastinfoset,application/bjjson,application/x-google-protobuf"/>
        <Engine name="Catalina" defaultHost="localhost">
            <Realm className="org.apache.catalina.realm.LockOutRealm">
                <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                       resourceName="UserDatabase"/>
            </Realm>
            <Host name="localhost"  appBase="empty"
                  unpackWARs="true" autoDeploy="true">
                <Context path="${vdir.server}" docBase="${docbase.server}" reloadable="false"/>
                <Valve className="org.apache.catalina.valves.AccessLogValve" directory="${log.server}"
                       prefix="access." suffix="log" rotatable="true" renameOnRotate="true" fileDateFormat="yyyy-MM-dd.HH." maxDays="1"
                       pattern="^A%t^A%{Host}i^A%v^A%A^A%m^A&quot;%r&quot;^A%U^A%q^A%p^A%u^A%a^A%{X-Forwarded-For}i^A%H^A&quot;%{User-Agent}i&quot;^A%{Referer}i^A%s^A%B^A%D^A&quot;%{rootmessageid}o&quot;^A&quot;%{x-request-id}i&quot;^A&quot;%{x-request-id}o&quot;"/>
            </Host>
        </Engine>
    </Service>
</Server>
```

## jvm

```perl
-XX:-UseZGC -XX:+UseG1GC
```

## startup.sh

```bash
#!/bin/bash
source /opt/app/setenv.sh
java $JAVA_OPTS -jar /opt/app/unicorn-0.0.1-SNAPSHOT.jar
```

## extraenv.sh

```perl
JAVA_OPTS="$JAVA_OPTS -Xmx2576m -Xms2576m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"
```

## setenv.sh

```bash
#!/bin/bash

# 内存 1G, 堆内 650M, 堆外 374
# 内存 2G~4G，堆外 800M
# 内存 6G 及以上，堆外占 25%

JVM_MAX_MEM=${CDOS_MEM:-1024}
CDOS_JVM_MIN_HEAP_FREE_RATIO=${CDOS_JVM_MIN_HEAP_FREE_RATIO:-40}
CDOS_JVM_MAX_HEAP_FREE_RATIO=${CDOS_JVM_MAX_HEAP_FREE_RATIO:-70}

if [ "$env" = "PROD" ]; then
  CDOS_JVM_XMS2XMX_RATIO=${CDOS_JVM_XMS2XMX_RATIO:-100}
else
  CDOS_JVM_XMS2XMX_RATIO=${CDOS_JVM_XMS2XMX_RATIO:-60}
fi

if [ $JVM_MAX_MEM -eq 1024 ]
then
  RESERVED_MEM=374
elif [ $JVM_MAX_MEM -eq 2048 ] || [ $JVM_MAX_MEM -eq 3072 ] || [ $JVM_MAX_MEM -eq 4096 ]
then
  RESERVED_MEM=800
elif [ $JVM_MAX_MEM -ge 6144 ]
then
  RESERVED_MEM=$(echo "$JVM_MAX_MEM * 0.25" | bc -l)
  RESERVED_MEM=${RESERVED_MEM%.*} # 取整
fi

JVM_AGG_MAX_MEM=$(echo "$JVM_MAX_MEM - $RESERVED_MEM" | bc -l)
JVM_AGG_MIN_MEM=$(echo "$JVM_AGG_MAX_MEM * $CDOS_JVM_XMS2XMX_RATIO / 100" | bc -l)
JVM_AGG_MIN_MEM=${JVM_AGG_MIN_MEM%.*} #取整

JVM_GC_THREADS=${CDOS_CPUS:-1}
JVM_GC_THREADS=${JVM_GC_THREADS%.*}

if [ $JVM_GC_THREADS -lt 1 ]; then
  JVM_GC_THREADS=1
fi

#向上取整
JVM_ActiveProcessorCount=$((${CDOS_CPUS//.*/+1}))

if [ $JVM_ActiveProcessorCount -lt 1 ]; then
  JVM_ActiveProcessorCount=1
fi

SERVER_LOG="/opt/logs/$app_id"

if [ "$SERVER_VDIR" = "/" ]; then
  SERVER_VDIR=""
fi

export LC_ALL="en_US.UTF-8"

JMX_HOSTNAME=$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

JAVA_OPTS="-server \
           -Xmx${JVM_AGG_MAX_MEM}m \
           -Xms${JVM_AGG_MIN_MEM}m \
           -Xss256k \
           -XX:+UseZGC \
           -XX:MetaspaceSize=128m \
           -XX:MaxMetaspaceSize=256m \
           -XX:+UnlockExperimentalVMOptions \
           -XX:MinHeapFreeRatio=${CDOS_JVM_MIN_HEAP_FREE_RATIO} \
           -XX:MaxHeapFreeRatio=${CDOS_JVM_MAX_HEAP_FREE_RATIO} \
           -Djava.util.concurrent.ForkJoinPool.common.parallelism=${JVM_GC_THREADS} \
           "

if [ -r "/opt/app/extraenv.sh" ]; then
  . "/opt/app/extraenv.sh"
fi

JAVA_OPTS="${JAVA_OPTS} \
           -XX:ActiveProcessorCount=${JVM_ActiveProcessorCount} \
           -Xlog:gc*:file=${SERVER_LOG}/gc.log:tags,time,uptime,pid:filecount=5,filesize=32M \
           -XX:+HeapDumpOnOutOfMemoryError \
           -XX:-OmitStackTraceInFastThrow \
           -XX:HeapDumpPath=${SERVER_LOG}/java-${CDOS_POD_NAME}.hprof \
           -XX:OnOutOfMemoryError=/opt/container/tools/heapdump_upload.sh \
           -Djava.security.egd=file:/dev/./urandom \
           -Djava.rmi.server.hostname=${JMX_HOSTNAME} \
           -Dcom.sun.management.jmxremote \
           -Dcom.sun.management.jmxremote.port=8780 \
           -Dcom.sun.management.jmxremote.rmi.port=8780 \
           -Dcom.sun.management.jmxremote.authenticate=false \
           -Dcom.sun.management.jmxremote.ssl=false
           -Djdk.attach.allowAttachSelf=true \
           -XX:+EnableDynamicAgentLoading \
           --add-opens=java.management/sun.management=ALL-UNNAMED \
           --add-opens=java.management/java.lang.management=ALL-UNNAMED \
           --add-opens=java.base/java.lang=ALL-UNNAMED \
           --add-opens=java.base/java.lang.reflect=ALL-UNNAMED \
           --add-opens=java.base/sun.reflect.annotation=ALL-UNNAMED \
           --add-opens=java.base/java.math=ALL-UNNAMED \
           --add-opens=java.base/java.util=ALL-UNNAMED \
           --add-opens=java.base/sun.util.calendar=ALL-UNNAMED \
           --add-opens=java.base/java.io=ALL-UNNAMED \
           --add-opens=java.base/java.net=ALL-UNNAMED \
           --add-opens=java.base/java.util.concurrent=ALL-UNNAMED \
           --add-opens=java.xml/com.sun.org.apache.xerces.internal.jaxp.datatype=ALL-UNNAMED \
           --add-opens=jdk.internal.jvmstat/sun.jvmstat.monitor=ALL-UNNAMED \
           -XX:+ZGenerational \
           "

rm -rf ${SERVER_LOG}/gc.log*-*
for f in ${SERVER_LOG}/gc.log*; do
   if [[ -r "$f" ]]
   then
     TIMESTAMP=$(date +%Y%m%d%H%M%S)
     mv "$f" "$f-${TIMESTAMP}"
   fi
done

unset LD_PRELOAD

export JAVA_OPTS=${JAVA_OPTS}
```

# 超时治理

**背景：**

近几年，出现比较多起由于超时配置不合理的RCA事件。

**超时治理原因**

服务都有⾃身的硬件资源上限，直观来看，响应时间会对资源消耗产⽣直接影响。

服务消费方

- 协议消耗，每次发起 TCP 连接请求时，通常会让系统选取⼀个空闲的本地端⼝（local port），该端⼝是独占的，不能和其他 TCP 连接共享。TCP 端⼝的数据类型是 unsigned short，因此可⽤端⼝最多只有 65535，所以在全部作为 client 端的情况下，最⼤TCP 连接数 65535。
- 除端⼝消耗外，发起调⽤的业务进程、线程、协程等待过程中，⽆法释放其所消耗的内存、CPU 等，是服务消费⽅发起调⽤的主要消耗。

在调⽤持续发⽣且服务提供⽅不及时返回的情况下，未触发性能拐点前，可以简化认为资源的消耗是线性增⻓。微服务发起⼀个请求，会占⽤⼀个空闲的本地端⼝，当然，每个连接所对应的业务处理过程，也会对应消耗内存、IO、CPU 消耗等资源，

- 资源剩余数（RR, Resource Remaining），将受这三个变量影响：每秒请求数（QPS）、请求持续时间 （Duration ）和资源释放的速度（ Release *Tinny_T）

​    从资源静态和动态分析看，我们应当规范设置调⽤超时时间。设置超时时间的意义， 是在极端情况下，采⽤主动的快速失败策略，使得资源消耗与释放资源之间达到平衡，避免调⽤双⽅因资源耗尽⽽宕机。超时时间设置不当引发，容易引发⽣产故障， 线上已经有诸多血的教训。

设置过长：

超时时间过⻓容易引起降级失效、系统崩溃、连接池爆满等问题。

设置过短：

超时时间设置过短，实际⽣产中容易因⽹络抖动⽽告警频繁，造成服务不稳定等⽤户体验问题。

**推荐值：**

| **客户端APPID** **(可以考虑先JAVA)** | **目标类型** | **目标KEY（应用、OP、DB、Redis)** | **SDK版本** **(不同版本的默认超时配置不同）** | **connection-request-timeout** | **connecttimeout** | **sockettimeout （当前配置）** **建议按照实际响应时间的P95/P99来设置** | **实际响应时间的P95/P99** | **推荐值** | **操作**         |
| ------------------------------------ | ------------ | --------------------------------- | --------------------------------------------- | ------------------------------ | ------------------ | ------------------------------------------------------------ | ------------------------- | ---------- | ---------------- |
|                                      | SOA          | international.getlocaleconfig     | 2.21.2                                        | 200ms                          | 1200ms             | 100s                                                         | 5s                        | 5s         | 跳monkey专项演练 |

**合理设定规则**

合理设置超时时间，对系统的稳定来⾔，非常重要，我们可以从以下⻆度来考虑设置值：

⽤户⻆度：服务最终为⽤户提供服务，从⽤户交互数据来讲，服务响应时间在300ms内最佳， 2000ms内仍可接受。通常情况下，建议超时的上限值为2000ms，超过2000ms的非重要请求，则有必要被降级处理。 

技术⻆度：同时考虑到TCP算法中 Delayed ACK + Nagle算法开启的场景，最小delay值 40 ms，建议下限值设定为 50ms； 在RTT较⼩的正常网络环境中，TCP数据包丢包，超时重传的最⼩值，200 ms，因此我们建议 300ms 可以视为超时设置的最佳选择，为重传保留⼀定的余量。

资源消耗⻆度：依据资源消耗的分析，超时时间⻓短应当和QPS成反⽐例。我们设定基础值超时设定为 300ms、100QPS，并根据实际 QPS做调整。

**超时配置类型**

| 配置类型                             | 超时类别                                                     | 说明                                                         | 建议参数（ms）                                   | 默认参数（ms）                                   | 后期更改                                                     | *备注*                                                       |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SOA2Client.Timeout（java）           | connectTimeOut                                               | 和服务端建立连接超时客户端底层是 Apache HTTP Client，采用连接复用，底层维护了一个连接池，客户端在发请求时，需要从连接池获取获取一个可用连接如果连接池内没有空闲连接，并且连接池内连接数没有达到最大连接数，那么客户端会创建一个新连接ConnectTimeout 就是作用于此时与服务端建立连接的超时时间。如果在设定的 timeout 时间内，未能成功连接到服务器，会抛出 ConnectTimeoutException。 | 1200                                             | 1050                                             |                                                              |                                                              |
| ConnectionPoolBorrow                 | 和应用程序池建立连接超时客户端底层是 Apache HTTP Client，采用连接复用，底层维护了一个连接池，客户端在发请求时，需要从连接池获取获取一个可用连接如果连接池内连接数已达到最大值，此时需要等待一个空闲连接，ConnectionRequestTimeout 就是作用于这个等待空闲连接的超时时间从 connection pool 获取一个 connection 最大等待时间。如果在 timeout 时间内没有获取 connection，会抛出 ConnectionPoolTimeoutException。 | 200                                                          | 200                                              |                                                  |                                                              |                                                              |
| socketTimeOut                        | 和服务端包传输超时每次在 Socket 上调用 write/read 的超时时间，一个请求周期内，可能会在 Socket 上进行多次 write/ read，每次都重新计算超时时间，也就是说这不是一个累加的超时时间。这意味着 **调用总时间 不一定 总是小于 Socket Timeout** 。在请求全部发到服务端前，作用于客户端在 Socket 上调用 write 的超时时间。 在请求全部发到服务端后，客户端会在 Socket 上调用 read 方法，此时作用于在 Socket 上接收到响应的第一个字节（ TTFB ），此时且仅限于此时 Socket Timeout 是服务端业务的处理的超时时间。在客户端接收到响应的第一个字节后，作用于客户端后续在 Socket 上调用 read 的超时时间。 注意：如果请求或响应很大或者网络不好（ 频繁丢包/重传 ）时，客户端可能会调用 write/read 很多次，此时会出现实际调用时间远大于 Socket Timeout 时间的情况。 | 服务端最大响应时间*1.4                                       | 4000                                             |                                                  |                                                              |                                                              |
| SOAClient.Timeout（nodejs）          | requestTimeout                                               | 请求级别超时参数，表示建立连接和读写的总耗时                 |                                                  | 如果用户没有配置客户端超时需要依赖服务端120s超时 |                                                              |                                                              |
| serviceTimeout                       | 服务级别超时参数，表示建立连接和读写的总耗时                 |                                                              | 如果用户没有配置客户端超时需要依赖服务端120s超时 |                                                  |                                                              |                                                              |
| credis（java）                       | connectTimeOut                                               | 和服务端建立连接超时                                         |                                                  | 300                                              |                                                              |                                                              |
| Pool Wait Timeout                    | 和连接池建立连接超时                                         |                                                              | 250                                              |                                                  |                                                              |                                                              |
| socketTimeOut                        | 和服务端包传输超时                                           |                                                              | 800                                              |                                                  |                                                              |                                                              |
| credis（nodejs）                     | `connectionTimeout`                                          | 和服务端建连超时                                             |                                                  | 300                                              |                                                              | 当前配置>1500(ms)，label=1，推荐值=300(ms)                   |
| DAL                                  | queryTimeout                                                 | 查询超时                                                     |                                                  | 60000                                            |                                                              |                                                              |
| connectTimeOut                       |                                                              |                                                              | 1200                                             |                                                  |                                                              |                                                              |
| socketTimeOut                        |                                                              |                                                              | 100000                                           |                                                  |                                                              |                                                              |
| connectionPoolBorrowTimeout          |                                                              |                                                              | 4000                                             |                                                  |                                                              |                                                              |
| removeAbandonedTimeout               | 泄露的连接可以被移除的超时时间                               |                                                              | 65000                                            |                                                  |                                                              |                                                              |
| cmongo                               | clusterSettings.serverSelectionTimeoutMS                     | 选择可用节点的最大等待时间                                   |                                                  | 2000                                             |                                                              |                                                              |
| socketSettings.connectTimeoutMS      | socket建连超时时间                                           |                                                              | 1200                                             |                                                  |                                                              |                                                              |
| socketSettings.readTimeoutMS         | socket读超时时间                                             |                                                              | 60000                                            |                                                  |                                                              |                                                              |
| connectionPoolSettings.maxWaitTimeMS | 连接池等待连接超时时间                                       |                                                              | 4000                                             |                                                  |                                                              |                                                              |
| dubbo                                | socketTimeOut                                                | 和服务端包传输超时                                           | 读取soa配置参数                                  |                                                  |                                                              |                                                              |
| es                                   | connectTimeOut                                               | 和服务端建立连接超时                                         |                                                  | 1200                                             |                                                              | [ES 客户端动态超时时间-用户文档#%E4%BF%AE%E6%94%B9%E8%B6%85%E6%97%B6%E9%85%8D%E7%BD%AE](http://conf.com/pages/viewpage.action?pageId=2804717116#ES客户端动态超时时间用户文档-修改超时配置) |
| socketTimeOut                        | 和服务端包传输超时                                           |                                                              | 1000                                             |                                                  |                                                              |                                                              |
| ConnectionRequestTimeout             | 和应用程序池建立连接超时                                     |                                                              | 1000                                             |                                                  |                                                              |                                                              |
| hbase                                |                                                              |                                                              |                                                  |                                                  | hbase在本地实现快速熔断效果，预计sdk在流浪地球考试完成后开始推广 | 三、合理设定规则                                             |

# 闲置资源治理

### 闲置检测逻辑汇总

| 项目                     | 检测维度                   | 判断逻辑                                                     | 检测目的                                                     | 处理建议                                                     |
| :----------------------- | :------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.生产环境闲置server检测 | 单机维度主键endpoint       | 1.grouptype为server类型；(citype=SVR/VMS)2.appContainer符合以下类型：['windows_web_iis', 'linux_tomcat', 'linux_java', 'linux_common', 'linux_apache_php', 'linux_nginx_nodejs', 'linux_webresource', 'linux_nginx_go', 'linux_nginx_php', 'windows_common', 'windows_clr', 'linux_squid', 'linux_python', 'custom', 'windows_jump','linux_go','linux_clickhouse']3.建立时间大于90天，最近90天没有发布4.单台server最近30天没有请求；5.单台server最近30天的cpu利用率过低（cpu均值和cpu标准差都非常低）；6.最近30天没有登录；满足以上条件判断为可能闲置server。 | 检测已经不提供服务的单机检测范围：1）docker以外的虚拟机和物理机2）含公有云3）表格内增加一列region-idc | 服务器资源使用很低，没有使用迹象，建议用户做下线处理         |
| 2.aws闲置region检测      | region维度主键appid+region | 1.grouptype为docker类型；且不是mirror or test2.appContainer符合以下类型：['windows_web_iis', 'linux_tomcat', 'linux_java', 'linux_common', 'linux_apache_php', 'linux_nginx_nodejs', 'linux_webresource', 'linux_nginx_go', 'linux_nginx_php', 'windows_common', 'windows_clr', 'linux_squid', 'linux_python', 'custom', 'windows_jump','linux_go','linux_clickhouse']3.region类型为SIN（SIN-AWS）或FRA（FRA-AWS）4.聚合维度过去30天无流量/流量过低5.聚合维度最近30天的cpu利用率过低（cpuavg<3 and cpustd<0.6）满足以上条件判断为资源利用率低region。 | 检测不需要上云的应用                                         | 应用已经在海外部署（海外计算资源成本比较高），但是判断应用相关region并没有开始使用，建议用户如果在上云过程中加快上云部署和流量切换的进度，否则容器暂停使用（缩容到0台），等完成开发和测试环境再进行云上部署。 |
| 3.闲置docker应用检测     | group维度主键groupid       | 1.grouptype为docker类型；2.非mirror, citype = DKR,  *idc='SHARB' or idc='SHAXY' or idc='SHA-ALI'*3.appContainer符合以下类型：['windows_web_iis', 'linux_tomcat', 'linux_java', 'linux_common', 'linux_apache_php', 'linux_nginx_nodejs', 'linux_webresource', 'linux_nginx_go', 'linux_nginx_php', 'windows_common', 'windows_clr', 'linux_squid', 'linux_python', 'custom', 'windows_jump','linux_go','linux_clickhouse']4.建立时间大于90天，且最近90天没有发布5.group维度过去30天流量过低6.group维度最近30天的cpu利用率过低（cpuavg<0.4 and cpustd<0.4）7.增加一个逻辑是否该应用的所有SHA group都闲置（从而和“aws闲置region检测合并”） | 检测范围：1) docker应用2）当前仅含sharb\shaxy\sha-ali        |                                                              |
| 4.闲置nlb检测            | 实例维度主键instanceId     | 检测公有云上NLB所有实例状态，活跃连接数一周加总小于100，即判断为可能闲置。 | 检测闲置nlb实例                                              | 如果长时间没有流量的集群建议下线，某些属于测试用途需要保留，最好在集群上有标识可以在巡检中做过滤。否则在后期的ido工具中申请白名单定期review |
| 5.测试环境闲置资源检测   |                            | 判断逻辑 同1和3                                              |                                                              |                                                              |

备注：

sin/fra三字码新标准，sin-aws/fra-aws六字码老标准

ido：

[http://ido.com/customs?type=%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E7%A9%BA%E9%97%B2%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%B2%BB%E7%90%86](http://ido.com/customs?type=生产环境空闲服务器治理)

[http://ido.com/customs?type=%E4%BD%8E%E8%B4%9F%E8%BD%BD4%E5%B1%82%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E9%9B%86%E7%BE%A4%E6%B2%BB%E7%90%86](http://ido.com/customs?type=低负载4层负载均衡集群治理)

gitlab服务 40C 128G 2T *15台