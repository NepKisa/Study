Grafana

使用监控，需要先有数据，所以从制造数据开始，此处称之为接入hickwall

# Hickwall接入

通过调用SDK，在应用中初始化不同的计数器，然后由一个线程reportor定时地将这些计数器的状态汇报给hickwall。

适用于自研组件（应用）的监控，要求必须有appid，hickwall将对每个appid的接入数据量进行监控和限流

> ## 常用埋点
>
> 常用的埋点主要是这三种：Counter、ResetCounter、Histogram。
>
> 使用第四代存储引擎VM
>
> 1. 只关心应用的QPS，建议使用Counter做请求量监控
> 2. 想要长时间的报表，例如一周总的请求量，建议使用ResetCounter
> 3. 关心某一类请求的响应时间使用Histogram，通过p99、p95来监控
>
> #### 注意[metrics埋点不是日志埋点]
>
> #### 严格禁止使用 uid requestid 订单号 时间戳 hash值等作为埋点的tag
>
> #### tag的value必须是长期存在的值，不能是像上面那种唯一值，需要是枚举值，tag value过多会影响查询性能[因为需要建索引]
>
> #### tag的value的字符内容不能过长，最大为128个字符，值种类不超过3000
>
> #### 指标名(measurement)和tag名称不能使用中文，必须以英文字母开头，最好用下划线来连接字符
>
> #### 非法的指标写入会被监控到并且被限流，用户会收到限流邮件，需要改正埋点

## UDP方式接入 [适用于JAVA应用]

这种方式适合于应用内埋点，增加了对Spring支持

添加依赖，版本由bom控制，无需关心

```xml
<dependency>
  <groupId>com.ctrip.ops</groupId>
  <artifactId>hickwall-sdk</artifactId>
</dependency>
```

监控工具类【==BU 必须使用OPS==】

```java
import com.ctrip.ops.hickwall.HickwallUDPReporter;
import io.dropwizard.metrics5.MetricRegistry;

import java.util.concurrent.TimeUnit;

/**
 * @author Nepkisa
 * @date 2024/7/16 15:59
 */
public class Monitor {
    // 初始化一次
    public static final MetricRegistry metrics = new MetricRegistry();


    static {
        HickwallUDPReporter.enable(
                metrics,            // singleton metrics registry
                60,                 // interval, minimum 10s
                TimeUnit.SECONDS,   // interval time unit
                "OPS"              // BU CODE，例如OPS
        );
    }

}
```

基础埋点【此处统计应用请求量】，measurement为neptune.unicorn.request.solution.count，field固定返回count，故指标名为neptune.unicorn.request.solution.count_count

```java
import com.ctrip.it.neptune.unicorn.service.impl.DiffrerntPathServiceImpl;
import com.ctrip.it.neptune.unicorn.utils.Monitor;
import io.dropwizard.metrics5.Counter;
import io.dropwizard.metrics5.MetricName;
import io.dropwizard.metrics5.MetricRegistry;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

/**
 * @author Nepkisa
 * @date 2024/1/11 18:22
 */
@RequestMapping("/it")
@RestController
public class DiffrerntPathController {

    @Autowired
    DiffrerntPathServiceImpl service;

    @RequestMapping("/solution")
    public Integer soultion(Integer m, Integer n) {
        MetricRegistry metrics = Monitor.metrics;
        // 非全局的tag
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        Counter counter = metrics.counter(new MetricName("neptune.unicorn.request.solution.count", tags));
        counter.inc();    //+1,  请求数加一
        return service.soultion(m, n);
    }
}
```

启动程序后，发起请求即可制造并写入数据到数据库

查询数据：https://hickwall.qa.nt.ctripcorp.com/pd/#/metrics/series

数据源选择APM-OPS，搜索指标名即可，点击指标可以查看详情

==注意：metric=measurement_field==

![image-20240716203707306](D:/Code/Notes/devops/hickwall/HickWall.assets/image-20240716203707306.png)

选择时间后可以看到原始数据，点击生成看板到Grafana即可自动生成Panel 

![image-20240716203814944](D:/Code/Notes/devops/hickwall/HickWall.assets/image-20240716203814944.png)

 https://bat.fws.qa.nt.ctripcorp.com/d/2G1aSRXSk/unicorn?orgId=0

![image-20240716204128930](D:/Code/Notes/devops/hickwall/HickWall.assets/image-20240716204128930.png)



以上试验了最简单的数据，验证数据可以在看板上读取到，接下来准备制造更复杂的数据

```java
   /**
     * ResetCounters (计数器)
     * 表示每个汇报周期的请求量，sdk向hickwall汇报完数据后自动清零
     * 最后返回的field名为rcount
     */
    private void resetCounterMetrics() {
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        tags.put("type", "ResetCounters");
        ResetCounter rcounter = metrics.resetCounter(new MetricName("neptune.unicorn.solution.request", tags));//.setZero(true),请求量为0也向hickwall汇报，默认不汇报
        rcounter.inc(); // 加1
        rcounter.inc(2); // 加2
        rcounter.dec(); // 减1
        rcounter.dec(2); // 减2
        rcounter.inc();
    }

    /**
     * Counters （累加计数器）
     * 表示一个不断递增的值
     * 最后返回的field名为count
     */
    private void counterMetrics() {
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        tags.put("type", "Counters");
        Counter counter = metrics.counter(new MetricName("neptune.unicorn.solution.request", tags));
        counter.inc();    //+1,  请求数加一
    }

    /**
     * ResetTimer（响应时间分布度量器）
     * 表示时间的间隔，多用记录执行时间
     * 最后返回的field有：
     * <p>
     * rcount 单位时间请求量
     * mean 平均值
     * p95 95值
     * p99 99值
     * p999 999值
     * m1_rate 汇报周期内的QPS
     */
    private void resetTimerMetrics() {
        // 非全局的tag
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        tags.put("type", "ResetTimer");
        ResetTimer resetTimer = metrics.resetTimer(new MetricName("neptune.unicorn.solution.rtimer", tags));//.setZero(true),请求量为0也向hickwall汇报，默认除rcount之外不汇报
        resetTimer.update(10, TimeUnit.MILLISECONDS);
    }


    /**
     * Gauges（瞬态值度量器）
     * 表示一个特定的值
     * 最后返回的field名为value
     */
    private void gaugesMetrics(int m) {
        MetricName metricName = MetricName.build("neptune.unicorn.solution.gauges");
        AtomicInteger atom = new AtomicInteger(0);
        metrics.gauge(metricName, () -> () -> atom.getAndSet(0));
        atom.getAndSet(m);
    }

    /**
     * Meters（吞吐率度量器）
     * 统计频率
     * 最后返回的field为m1_rate
     * m1_rate，表示最近一分钟内平均每秒发生次数，这是一个大概的统计值，不是精确值。
     */
    private void metersMetrics() {
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        tags.put("type", "Meters");
        Meter meter = metrics.meter(new MetricName("neptune.unicorn.solution.meters", tags));
        meter.mark(); // 加1
    }

    /**
     * Histograms（直方分布度量器）
     * 统计分布情况
     * 最后返回的field有：
     * <p>
     * mean 平均值
     * p95 95值
     * p99 99值
     * p999 999值
     */
    private void histogramsMetricsN(int n) {
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        tags.put("type", "Histograms");
        tags.put("mertic", "n");
        Histogram histogram = metrics.histogram(new MetricName("neptune.unicorn.solution.histograms.n", tags));
        histogram.update(n);
    }

    private void histogramsMetricsM(int m) {
        Map<String, String> tags = new HashMap<>();
        tags.put("appid", "100054096");
        tags.put("method", "soultion");
        tags.put("type", "Histograms");
        tags.put("mertic", "m");
        Histogram histogram = metrics.histogram(new MetricName("neptune.unicorn.solution.histograms.m", tags));
        histogram.update(m);
    }
```

默认jvm参数

```bash
-Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xmx3296m -Xms1977m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -XX:MinHeapFreeRatio=30 -XX:MaxHeapFreeRatio=50 -Djava.util.concurrent.ForkJoinPool.common.parallelism=2 -XX:ActiveProcessorCount=2 -Xlog:gc*:file=/opt/logs/100053708/gc.log:tags,time,uptime,pid:filecount=5,filesize=32M -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -XX:HeapDumpPath=/opt/logs/100053708/java-r100053708-71109524-ghf5n.hprof -XX:OnOutOfMemoryError=/opt/container/tools/heapdump_upload.sh -Dport.http.server=8080 -Dlog.server=/opt/logs/100053708 -Dport.shutdown.server=8081 -Ddocbase.server=/opt/app -Dvdir.server= -Djava.security.egd=file:/dev/./urandom -Djava.rmi.server.hostname=10.4.23.22 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8780 -Dcom.sun.management.jmxremote.rmi.port=8780 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -DAPPLOGDIR=/opt/logs/100053708/applog -Djdk.attach.allowAttachSelf=true -XX:+EnableDynamicAgentLoading --add-opens=java.management/sun.management=ALL-UNNAMED --add-opens=java.management/java.lang.management=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/sun.reflect.annotation=ALL-UNNAMED --add-opens=java.base/java.math=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/sun.util.calendar=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.xml/com.sun.org.apache.xerces.internal.jaxp.datatype=ALL-UNNAMED --add-opens=jdk.internal.jvmstat/sun.jvmstat.monitor=ALL-UNNAMED -XX:+ZGenerational -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED -Dcatalina.base=/opt/tomcat -Dcatalina.home=/opt/tomcat -Djava.io.tmpdir=/opt/tomcat/temp
```

## 数据类型

| 指标类型   | 统计方式（filed） | 说明                                     |
| ---------- | ----------------- | ---------------------------------------- |
| count      | rcount            | 总次数（一分钟内的总次数）               |
| time和size | m1_field          | 平均每秒的次数（一分钟内平均每秒的次数） |
| time和size | rcount            | 总次数（一分钟内的总次数）               |
| time和size | rsum              | 总和（一分钟内的总和）                   |
| time和size | mean              | 每分钟的均值（一分钟内的rsum除以rcount） |
| time和size | p95/p99/p999      | P值                                      |
| value      | value             | 采样时的实时值                           |

数据值，json格式

```json
{
      "metric": {
        "hostname": "r100054096-71112160-pjgcc",
        "route": "/it/solution",
        "method": "DiffrerntPath",
        "sourceIP": "10.118.237.224",
        "__name__": "neptune.unicorn.solution.request_count",
        "appid": "100054096",
        "groupId": "71112160",
        "ip": "10.118.237.224",
        "idc": "NTGXH",
        "env": "FAT",
        "type": "Counters",
        "~db": "APM-OPS"
      },
      "values": [
        [
          1722413432.744,
          "104"
        ],
        [
          1722413442.744,
          "106"
        ],
        [
          1722413452.744,
          "108"
        ],
        [
          1722413462.744,
          "110"
        ],
        [
          1722413472.744,
          "112"
        ],
        [
          1722413482.744,
          "114"
        ]
      ]
    }
```

原数据

```bash
* Tags： __name__=neptune.unicorn.solution.request_count appid=100054096 env=FAT groupId=71112160 hostname=r100054096-71112160-pjgcc idc=NTGXH ip=10.118.237.224 method=DiffrerntPath route=/it/solution sourceIP=10.118.29.34 type=Counters ~db=APM-OPS
DateTime	TimeStamp	Value
2024-07-31 16:10:32	1722413432.744	17424
2024-07-31 16:10:42	1722413442.744	17667
2024-07-31 16:10:52	1722413452.744	17900
2024-07-31 16:11:02	1722413462.744	18874
2024-07-31 16:11:12	1722413472.744	19500
2024-07-31 16:11:22	1722413482.744	19617

* Tags： __name__=neptune.unicorn.solution.request_count appid=100054096 env=FAT groupId=71112160 hostname=r100054096-71112160-pjgcc idc=NTGXH ip=10.118.237.224 method=DiffrerntPath route=/it/solution sourceIP=10.120.205.144 type=Counters ~db=APM-OPS
DateTime	TimeStamp	Value
2024-07-31 16:10:32	1722413432.744	17558
2024-07-31 16:10:42	1722413442.744	17782
2024-07-31 16:10:52	1722413452.744	18014
2024-07-31 16:11:02	1722413462.744	19071
2024-07-31 16:11:12	1722413472.744	19679
2024-07-31 16:11:22	1722413482.744	19796

* Tags： __name__=neptune.unicorn.solution.request_count appid=100054096 env=FAT groupId=71112160 hostname=r100054096-71112160-pjgcc idc=NTGXH ip=10.118.237.224 method=DiffrerntPath route=/it/solution sourceIP=10.118.237.224 type=Counters ~db=APM-OPS
DateTime	TimeStamp	Value
2024-07-31 16:10:32	1722413432.744	104
2024-07-31 16:10:42	1722413442.744	106
2024-07-31 16:10:52	1722413452.744	108
2024-07-31 16:11:02	1722413462.744	110
2024-07-31 16:11:12	1722413472.744	112
2024-07-31 16:11:22	1722413482.744	114
```



# 语法问题

## sum_overtime

ps：面板上显示的13:03分的点位 其实是13:02:00~13:02:59的最后一个点位，以下数据举例，是13:02:50的数据

sum 和 sum_overtime，sum是直接取面板点位计算总和，此处仅计算13:02:50的数据(306+314+324)为13:03的数据

sum_overtime，以1m计算，会取13:02:00~13:02:59所有点位的数据，此处计算（158+1868+316+306）为13:03的数据，若以2m计算则13:04取13:02:00-13:03:59的总和，13:05取13:03:00-13:04:59的总和

```perl
* Tags： __name__=neptune.unicorn.solution.request_rcount appid=100054096 env=FAT groupId=71112160 hostname=r100054096-71112160-hmkjq idc=NTGXH ip=10.120.212.59 method=DiffrerntPath route=/it/solution sourceIP=10.120.205.144 type=ResetCounters ~db=APM-OPS
DateTime	TimeStamp	Value
2024-08-27 13:02:00	1724734920.079	158
2024-08-27 13:02:10	1724734930.079	1868
2024-08-27 13:02:40	1724734960.079	316
2024-08-27 13:02:50	1724734970.079	306
2024-08-27 13:03:00	1724734980.079	308
2024-08-27 13:03:10	1724734990.079	1864
2024-08-27 13:03:20	1724735000.079	310
2024-08-27 13:03:30	1724735010.079	306
2024-08-27 13:03:40	1724735020.079	178
2024-08-27 13:04:00	1724735040.079	876

* Tags： __name__=neptune.unicorn.solution.request_rcount appid=100054096 env=FAT groupId=71112160 hostname=r100054096-71112160-hmkjq idc=NTGXH ip=10.120.212.59 method=DiffrerntPath route=/it/solution sourceIP=10.119.139.24 type=ResetCounters ~db=APM-OPS
DateTime	TimeStamp	Value
2024-08-27 13:02:00	1724734920.079	164
2024-08-27 13:02:10	1724734930.079	1710
2024-08-27 13:02:40	1724734960.079	308
2024-08-27 13:02:50	1724734970.079	314
2024-08-27 13:03:00	1724734980.079	310
2024-08-27 13:03:10	1724734990.079	1868
2024-08-27 13:03:20	1724735000.079	314
2024-08-27 13:03:30	1724735010.079	312
2024-08-27 13:03:40	1724735020.079	172
2024-08-27 13:04:00	1724735040.079	872

* Tags： __name__=neptune.unicorn.solution.request_rcount appid=100054096 env=FAT groupId=71112160 hostname=r100054096-71112160-hmkjq idc=NTGXH ip=10.120.212.59 method=DiffrerntPath route=/it/solution sourceIP=10.118.29.34 type=ResetCounters ~db=APM-OPS
DateTime	TimeStamp	Value
2024-08-27 13:02:00	1724734920.079	168
2024-08-27 13:02:10	1724734930.079	1882
2024-08-27 13:02:40	1724734960.079	306
2024-08-27 13:02:50	1724734970.079	324
2024-08-27 13:03:00	1724734980.079	308
2024-08-27 13:03:10	1724734990.079	1864
2024-08-27 13:03:20	1724735000.079	312
2024-08-27 13:03:30	1724735010.079	306
2024-08-27 13:03:40	1724735020.079	172
2024-08-27 13:03:50	1724735030.079	2
2024-08-27 13:04:10	1724735050.079	5086

```

## 运算符

以下这段SQL有问题，不可以直接相加，会有聚合数据问题和时间戳问题，使用transfer可以解决，但是隐藏源数据时有bug（会把其他都隐藏掉），不建议使用

```sql
sys.cpu.util_percent_value{group="sakura_100054096",ip="10.119.118.139"}+ sys.cpu.util_percent_value{group="sakura_100054096",ip="10.119.118.138"}
```

正确的写法为：over_time保证子查询时间戳一致，再用聚合函数，保证子查询的数据分组条数一致

```sql
sum(sum_over_time(sys.cpu.util_percent_value{group="sakura_100054096",ip="10.119.118.139"}[1m])) + sum(sum_over_time(sys.cpu.util_percent_value{group="sakura_100054096",ip="10.119.118.138"}[1m]))
```

经验值保证分组数据条数一致即可

```sql
sum(sys.cpu.util_percent_value{group="sakura_100054096",ip="10.119.118.139"}) + sum(sys.cpu.util_percent_value{group="sakura_100054096",ip="10.119.118.138"})
```

## 根据变量选择不同的执行语句

由于查询不到数据的查询语句在面板上不展示，此时可以使用同一变量控制不同的tag以达到目的【需要all对应的值在tag中才行，其实可以随意选择一个tag的值来做到】，此法可以泛用

如：变量名为group，变量值为10.119.118.138，10.119.118.139，sakura_100054096（all）

```sql
-- 变量为sakura_100054096，才有数据展示，原理：使用错误变量值匹配tag以达到不展示的效果
avg(sys.fs.pfree_value{driver="rootfs",group=~"$group",ip="10.119.118.139"})*4/7+avg(sys.fs.pfree_value{driver="rootfs",group=~"$group",ip="10.119.118.138"})*3/7
-- 变量为10.119.118.138，10.119.118.139，才有数据展示，原理：使用错误变量值匹配tag以达到不展示的效果
avg(sys.fs.pfree_value{driver="rootfs",group=~"sakura_100054096",ip="$group"})
```

# Grafana

左右双Y轴：http://conf.ctripcorp.com/display/FRAM/1+Multiple+y-axes

上下Y轴：http://conf.ctripcorp.com/display/FRAM/2+Negative+Y

## Dashboard

### 标尺联动

General --> Panel options-->Graph tooltip

默认为default：显示当前panel的数值

Shared crosshair：显示当前panel的数值，并在其他panel上刻画出同一时间点的虚线

Shared Tooltip：显示当前panel的数值，并在其他panel上刻画出同一时间点的虚线与数值

### Annotations

详细参考：http://conf.ctripcorp.com/pages/viewpage.action?pageId=1237206054

使用场景：

* Annotations是一种在Panel面板上展示 注释的功能

* 在Dashboard中设置，提供开关选项，开关打开，整个Dashboard中的Panel均展示，不能针对某个特定Panel设置

* Annotations只对 带有时间时序的 曲线图生效

操作方法：

* 点击 看板的 Dashbaord Settings按钮 , 切换到Annotations 导航

* 点 Add annotation query

* 为annotations命名，本次以查看应用发布历史为例，取名为：Captain Release, 选择一个获取数据的来源，选择ES-ClickHouse 为例

* 将固定某个appid的语句复制到Query框中

* Color： 是用来设置当有注释时, Panel中展示的线的颜色

* 设置完成后，点击左侧的【Save Dashboard】

### Variables

详细参考：http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235306296

使用场景：预先定义一组变量，如查询一些hosts，后续在panel的查询语句中可以使用这些变量

> ==一旦使用变量，请务必将 查询语句中的 写成 key=~'$variable'==
>
> 配图语法中使用 `$ `后跟变量名表示使用模板变量的值进行替换，如图所示当变量值只选择了1个host: xxx 时，`host=~'$QueryHost'`等价于`host``='xxx';`
>
> 当template variable勾选了`Multi-Value`或`Include All Option`且在变量选择时选择了多个变量或All时，需要使用诸如`host=~'$QueryHost'` 的语法以替换多个值; 

操作方法：

* 在看板的右上方点击 Dashboard Settings

* 选择Variables tab, Add Variable

### Tags

详细参考：http://conf.ctripcorp.com/pages/viewpage.action?pageId=1237206039

使用场景：

* DashboardList :  适用于想将同一类型的看板放在一个列表中，进行分类 和 模块划分，方便查看，参考Dahsboard首页各BU导航点击进去的效果 ， 以平台研发看板为例：http://grafana.fx.ctripcorp.com/d/wtzaRXFZz/bbz?orgId=2
* AlertList :  适用于 将 包含某些的特定Tag 看板产生的告警统一进行查看。以机票 App主流告警为例：http://hickwall.ctripcorp.com/grafanav2/d/DjOMpFiWk/nav-flight-ji-piao-hickwalljian-kong?orgId=2&fullscreen&panelId=134
* Dashboard Link:  适用于 需要给看板设置跳转链接。 以Kafka Admin看板为例： [http://bat.fx.ctripcorp.com/d/Ky1acVrWk/kafka-admin-panels?orgId=1&refresh=1m](http://grafana.fx.ctripcorp.com/d/Ky1acVrWk/kafka-admin-panels?orgId=1&refresh=1m)

操作方法：

* 在看板右上角点击 Dahsboard Setttings

* General区域，Tags中手动输入 Tag名字，点击Add即可
* 一定记得点击 【Save Dashboard】

### Links

使用场景：在配置一个看板时，有时候需要直接在看板中，跳转到相关的看板，或者外部链接，Grafana提供了给整个Dashboard、Panel、Data 三种方式添加link，并可以自动将参数带过去的功能。

### Versions

可以随时进行版本回退

### Permission

**权限支持的粒度**

- 支持按照Dashboard粒度设置 可修改看板权限
- 支持按照Dashboard粒度设置 可查看面板数据的权限

**权限的角色划分**

- 面板权限维护人（Maintainer）
- 可编辑面板权限人（Editor）
- 可查看面板数据权限人（DataViewer）

角色权重大小： Maintainer > Editor > DataViewer

| 角色                               | 权限说明                                                     | 权限配置 | 修改面板 | 查看面板数据 |
| :--------------------------------- | :----------------------------------------------------------- | :------- | :------- | :----------- |
| 面板权限维护人（Maintainer）       | 面板权限维护人，一旦配置了角色，面板仅可以由这些角色 给别人赋予权限，此角色默认有：编辑看板、查看数据权限 | √        | √        | √            |
| 可编辑面板权限人（Editor）         | 看板可编辑权限人，此角色默认有：编辑看板、查看数据权限。未设置时，默认全部人可修改，一旦设置了某些人，则只允许这些人编辑看板、查看数据 |          | √        | √            |
| 可查看面板数据权限人（DataViewer） | 可查看面板数据，未设置时，默认全部人可查看数据。 一旦设定了某些人，仅这些人 和 上述的 Editor 可查看数据 |          |          | √            |

**权限配置步骤**

==注： 如果只设置了Maintainer，而不设置Editor ，则面板还是 全部人可以编辑； 如果想设置 仅部份人可以修改面板，务必设置Editor==

* 打开面板，在右上角的settings按钮

* 切换到Permission Tab，点击“成为Maintainer”，自动将自己作为面板的权限维护人

* 随后开始Add 不同的人作为 Editor 或者 Data Viewer

### Row

详细参考：http://conf.ctripcorp.com/pages/viewpage.action?pageId=1237206065

使用场景：监控看板若新建了很多个Panel，可以使用Row对Panel进行分类，且可以折叠。

操作方法：

* 在看板的右上方 点击按钮

* 选择【add a new row】

* 将自动生成一个row， 且默认将当前看板中的panel 加到row中

* 鼠标hover在row 上 ，可以点击row上的设置图标 编辑Row Title， 删除点击垃圾桶图标即可，放心： 删除Row，并不会删除掉Row中包含的Panel

* 在新建了多个Row之后，可以直接拖动Panel到不同的Row中

* 点右上角的保存按钮，保存你的看板。

### 库面板(Library Panel)

详细参考：http://conf.ctripcorp.com/pages/viewpage.action?pageId=1319964117

使用场景：在多个仪表板中重复使用（共享），对最原始的库面板 更改或更新将反映在使用该库面板的每个仪表盘上（统一更新）。

操作方法：

* Create Library Panel

    设置库面板的方式有两种： 

    * 针对已有的(创建好的)Panel，直接设置为 Library Panel
    * 新建一个Panel, 保存为 Library Panel

* Add a Panel from Library Panel

在New Dashboard时, 选择 Add a Panel from the panel library，弹出所有的Library list， 搜索或者选择你想要的，点击后，将自动加载图形效果, 再点击右上角保存看板即可；

* Unlink Library Panel

    如果你的看板中包含了库面板(也就是通过第2步新建)，若原始Library Panel进行了修改，您的看板也会随之修改, 若不想被共享修改, 可以在看板中 unlink，去除引用，方式有两种：

    * 直接在看板的Panel Header点击，下拉More - Unlink library panel（如下图1）
    * Panel Header点击，选择Edit， 在编辑页面的右上角 unlink（如下图2）

==注： 若在Edit页面中, 没有先 Unlink，对面板的任何修改，会将最原始的Library Panel修改掉，并应用于 任何引用过此Library Panel的Dashboard==

* Delete Library Panel（暂未开放Library删除能力）

删除一个库面板，请确保没有被用于任何Dashboard, 否则删除后, 引用过Library Panel看板中的数据也会被删掉；

### 设置面板自动刷新

说明：

- 面板自动刷新功能 最低设置为 1m，1是为了减轻对后端存储查询的压力，2是当一个面板panel过多时，太高频的自动刷新，会让上一次数据尚未加载完毕，就进行了下一次刷新；
- 设置了自动刷新的面板，如果包含了Clickhouse的数据源，查询时间范围不要过大，比如7d

操作方法：

* 新建面板并保存，不同插件配置面板的详细参考[03 【BAT Grafana】](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235311689)

* 在已经配置好的面板种，点击右上角的 Refresh Dashboard 图标旁边的 下拉箭头

## Panle

选择panel Edit后，右上角Visualizations可以选择图表类型

### Time Series Panel

Grafana原生插件， 默认且最常用的可视化插件； 属于趋势类分析图表，一般用于表示一组数据在一个有序数据类别（多为连续时间间隔）上的变化情况，用于直观分析数据变化, TimeSeries代替了原来的Graph插件。

#### Panel Options(Panel属性配置)

这里主要配置Panel的标题（Title）、描述（Description）、Panel透明度、Panel的外链（Panel links）配置（可以配置在描述下面 方便URL跳转）

**Panle links**

* Title是显示的文字
* URL是实际链接的地址
* Open in new tab（打开=点击链接时打开新的标签页）

**Repeat options 暂时未知用途**

#### Tooltip(鼠标经过图例展示的数据效果)

Single：表示只显示一条线的数值

All：表示显示所有的数据线数值，此时可以设置排序

* Ascending，正序排列 从小到大
* Descending，倒序排列 从大到小

Hidden：不显示数值

#### Legend(图例配置)

*Legend mode（图例模式）*：数据图例的展示方式

* List：默认平铺显示，数据实例：【max: 5,avg: 3】
* Table：表格显示，当需要聚合结果（max、avg）等，使用table更加方便查看，数据实例：【max、avg为表title】
* Hidden：不显示图例

*Legend placement（图例位置）*：线的名字展示的位置

* Bottom：底部显示，默认
* Right：右侧显示

*Legend values（图例显示值）*：计算值，可以为Last/Min/Max/Mean等等，线条上默认展示时间段内打点值

*  Min（最小值）| Max（最大值）| Mean（平均值）| Total（总计）| Last*（当前非空的值）| Last（当前值[含空值]）

#### Graph styles(绘图样式:线宽/透明度/空值/堆叠等)

*Style（整个时序图的展示方式 ）*

* Lines: 展示为线, 默认为线 
* Bars: 展示为竖的柱状
* Points: 展示为点

*Line interpolation（设置线的展示展示方式*）：分别为Line/Smooth/Step Before/Step After  [官方说明](https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/time-series/#line-interpolation)

*Line Width*：线的宽度(粗细) 可选数字 0-10 , 数字越大，线越粗

*Fill opacity*：填充对应的透明度, 数值是 0% ～ 100%

*Gradient mode（面板线条的渐变模式）*

* Opacity(使用Fill opacity的透明度作为面板的展示)
* Hue(面板颜色使用梯度色调显示)
* Schema(使用Thredsholds中的配置显示面板, 这个需要结合Threshold和Standard Options中的 Color schema 结合使用)
    * Gradient mode设置为 schema, 将Standard Options里的Color schema设置为 “From thresholds(by value)”,  将Thresholds中的阈值设置，并选择颜色

*Line style（线的展示方式）*

* Solid(实线) 
* Dash(虚线) 
* Dots(点线)

*Connect null values*：当值为空时的特殊处理

* Never为默认选项不处理
* Always表示将null连接起来
* Threshold可以指定连接的阈值

*Show Points*：是否显示数据点

*Point Size*：设置数据点的大小, 可设置数字 1～40

*Stack Series(堆叠)*： 可以使多条折线的数值叠加 

* Off：不开启堆叠，每条折线分别代表各自数据走势

* Normal：开启堆叠后折线的数值为多条折线数值的叠加。当指标数据为空时，是否要将线两个断掉的线连起来展示

* 100%：Y轴以百分比显示，最大百分比是100%

#### Axis(Y轴配置)

***Y轴通用配置***

Placement：Y轴显示位置（Auto/Left/Right/Hidden）

Label：Y轴标题

Width：Y轴宽度（width）

Soft min/max：弹性最大/小值

Show grid lines：面板上是否显示网格线

Scale：设置Y轴值的刻度

- Linear： 线性, 将刻度分成相等的部分。

- Logarithmic：使用对数刻度。当选择此选项时，会出现一个列表供选择二进制（基数为2）或常规（基数为10）对数刻度

***双Y轴配置***

激活双Y轴， 则需要搭配Overrides Tab去配置，在overrides中

- 点击按钮【+ Add field orverride】, 选择对应的指标名称
- 点击【Add override property】设置 Axis > Placement 设置为 Right
- 也可继续【Addsoverride property】，新增其他的属性，比如单位unit 等等

***Y轴显示负数***

将值显示为负数， 并将数据展示在负Y轴, 如通常网卡进出流量可以展示在正负Y轴上；

操作步骤：

* 切到【Overrides】Tab, 点击 + Add field overrides
* 可以选择, 可以根据自己需要去选择 获取的字段，这里以网络进的流量为例，选择了【Fields with name】
* 点击 + Add override property, 在弹出的框中输入“transform” 搜索, 选择 Graph styles > Transform ， 选择【Negative Y】 
* 保存看板

#### Standard options(单位/最大值/最小值/decimals)

Unit：单位

Min：Y轴最小值

Max：Y轴最大值

Decimal：保留小数点

Display name：指标名字特殊显示，比如写test，那么图上的IP就会变成test

Color schema：线的颜色选项

Color series by 这份只有选择了From thresholds才有

No value：空值或无数据的处理，填0表示null as zero

#### Data Links(数据链接)

详细参考：[03 【BAT Grafana】#03%E3%80%90BATGrafana%E3%80%91-3DataLinks](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235311689#id-03[BATGrafana]-03[BATGrafana]-3DataLinks)

使用场景：在面板数据point上点击，可以配置跳转的链接，既可以配置一个固定的链接，也可以配置 地址中 带上参数， 在链接地址里输入 $ 系统将会自动提醒

部分参数解释：

* Time range panel variables

    ```bash
    系统内置的全局时间变量
    
    ${__url_time_range} 表示 当前看板选择的时间范围 (如 ?from=now-6h&to=now)
    
    ${__from}  ${__to}  表示系统自带的全局时间范围，与上面的类似, 时间格式是unix格式
    ```

* Series variables：${__series.name} 表示指标的名称

* Template variables

    ```bash
    ${__all_variable}会把看板中配置的 variable 所有的参数都列出来
    
    ${group:queryparam}  可以指定一个参数
    
    注意${group:queryparam}  这样得出来的值 是 var-group=10023，如果只想拿到10023， 就写成${group:queryparam.value} 
    ```

* Fields： ${__data.fields[0]} 指的是当前语句查询的Fields的 Index来拿数据 从0 开始，一般 Time Series的 0 都是 time 字段，也可以指定特定的Field name的值

* Value： 查询语句返回的数据，并区分不同的单位

#### Value Mapping(值的映射)

#### Thresholds(阈值类型/阈值线显示)

可以设定数值阈值，阈值区域会用不同颜色区分。 

*Thresholds mode*

* Absoulte：数值
* Percentage：百分比

*Show thresholds*

* As lines：展示阈值线条
* As filled regions：展示阈值填充背景
* As filled regions and lines：展示阈值填充背景与线条

注：新版不支持按照Time Regions时间区域设置阈值显示

### Table Panel

详细参考：http://conf.ctripcorp.com/display/FRAM/02+Table+Panel

Table Panel是Grafana原生插件，可视化灵活，支持 时间序列和表格、注释和原始JSON 数据的多种模式。此面板还提供日期格式、值格式和着色选项， 新版本中增加了以下特性：

- 单元格支持 image 展示
- 单元格支持 值与进度条 展示 cell display mode as bar guage
- 表头检索列的内容

官方Demo： https://play.grafana.org/d/OhR1ID6Mk/3-table?orgId=1

#### Header and footer(表格头尾/分页)

Show header：显示首行标题

Show Footer：显示末行，开启后，Calculation里可设置计算方式，如Total

Enable pagination：开启分页

#### Table(列宽/筛选/对齐/列颜色)

Minimum column width：列的最小宽度

Column width：列宽

Column alignment：列的对齐方式

Cell display mode：单元格展示的样式

* 默认是Auto, 即没有任何样式的 正常 显示, 其他的选项 都和 Thresholds中的阈值线 强相关
* Tips: 这里的设置是针对所有列的单元格，如果需要针对某一个单元格，请去overrides中设置 单个字段的 cell display mode
* 这里控制了 单元格内的显示方式，若希望展示成 进度条，请选择Gauge相关，但是会将所有的列 都设置上。 若希望对某一列设置，请到Overrides中
    * 点击【+ Add field override】， 图例中的，平均数，设置成了bar guage的进度条模式，overrides 配置如下：Fields with name 设置为Mean，cell display mode设置程Gradient gauge

Cell value inspect：单元格数值检查

Column filter：开启列筛选

#### Transform

Series to rows：合并几个时间序列为一个。结果含三列：时间，字段名字(metric), 值

format as Table： 依据时间，将指标名变成多列, 一行多个指标，Table的插件默认就是 AS Columns的数据格式，针对Time Series To Columns 直接建议将 Formast as 配置为Table即可; 

Reduce- Series to rows ： 将每个查询出来的指标数据，转化为行, 表示 为每个字段（查询语句中的group by的tag）创建一行，为每个计算 (这里选择了Last* 和 Mean) 创建一个列。

Organize fields（修改title 列名）：此时的列名默认是聚合函数中的field，Last* 和 Mean, 若需要将重命名， 使用此函数

Sort by：若需要将表格中的数据，默认按照某一列进行排序，可使用此函数， 选择一个排序字段， 设置排序方式

### Stat Panel

详细参考：http://conf.ctripcorp.com/display/FRAM/03+Stat+Panel

Stat Panel是grafana的原生插件, grafana8.0 以上的版本，将原来的SingleStat和 MutilStat 合并成 Stat，同样可以使用阈值控制背景颜色或值颜色。区别是Stat Panel支持多值展示。

特性：Stat可以在一个Panel上添加多个值，相比SingleStat 添加效率要高，但需要是同一类的数据（因为单位不能分别设置）

官方Demo： https://play.grafana.org/d/Zb3f4veGk/2-stats?orgId=1

*对于stat只需要进行 数据配置、图形配置即可完成基础图形。*

**基本配置（Panel Options）**：标题、描述、Panel透明度、Panel的外链配置

**数据配置（Value options）**：值展示方式、最大返回行数、展示字段

- Calculation（计算），显示Stat的计算值，例如Calc设置为last、min或max时的值
- All Values（所有值），在指定查询时间范围内，显示Stat的所有值。

**图形配置（Stat Style）**：条形图布局位置（水平/垂直）、显示值设置、颜色设置

- Orientation（方向）：可设置为Horizontal（水平）或者Vertical（垂直）模式
- Color Mode（颜色）：可设置Value（值）或者Background（背景）颜色。Value表示仅对值和图形（sparklines）区域上色；Background表示给背景上色
- Graph Mode（图表）：None表示禁用图形/迷你图（sparklines）；Area graph表示显示低于值的面积图。这要求查询返回是基于时间序列的值
- Text alignmen（对齐方式）： Auto表示自动对齐，如果只显示一个值（不重复），则该值居中，如果有多个序列或行，则值为左对齐。Center表示居中

**文字大小（Text size）**：Title 大小，值大小

**标准配置（Standard options）**：单位、最大/小值设置、小数点位数、指标展示名、颜色方案

**数据链接（Data Links）**： 数值点上点击，可以外链

**阈值配置（Thresholds）**：阈值类型、阈值线显示

### Bar Gauge Panel

Bar gauge是Grafana的原生插件，可包含一个或多个条形图（即支持**一个语句**中 返回 单个或者多个值的展示），通常用于分析Top场景，直观地看到在单个值特定范围的占比，并展示类似条状图

官方更多配置： https://play.grafana.org/d/KIhkVD6Gk/4-gauges?orgId=1&refresh=10s

支持多种模式：进度条模式、LCD模式、渐变模式以及刻度盘模式，配合着不同的颜色渲染不同效果 ； Bar Gauge功能与 Gauge、Stat Panel功能相近，仅展示效果略有不同。 

**基本配置（Panel Options）**：标题、描述、Panel透明度、Panel的外链配置

**数据配置（Value options）**：值展示方式、最大返回行数、展示字段

- Show: 
    - All Values，表示在选定的时间范围内显示所有的采样值，如语句中的 index, count(1) 的值
    - Calculate，需要根据下方的Cal( Last, First,Min,Max等）中的具体选项来展示一个单一的数据，如，将只显示count(1)的值
- Limit: 显示值的最大上限
- Fields: 选择Numeric Fields，表示从计算结果中 显示数值的value

**图形配置（Bar gauge）**：条形图布局位置（Horizontal水平/垂直Vertical）、图形模式（LCD/进度条）、填充空置、最小高度

- Orientation: 柱状图水平或者垂直展示
- Display mode: 柱状图展示的风格样式, Gradient / Retro LCD(液晶显示格子) / Basic

**文字大小（Text size）**：Title 大小，值大小

**坐标轴配置（Axis）**：显示y轴、Y轴显示位置（Left/Right/Hidden）、y轴标题、Y轴宽度（width）、弹性最大/小值、网格线

**标准配置（Standard options）**：单位、最大/小值设置、小数点位数、指标展示名、颜色方案

- Color schema**:** 图表显示的样式与颜色。

**数据链接（Data Links）**： 数值点上点击，可以外链

**阈值配置（Thresholds）**：阈值类型、阈值线显示

### Gauge Panel

 Gauge是Grafana的原生插件，使用比较简单

BAT Demo：https://bat.fx.ctripcorp.com/d/7UnHhIZVz/03-bar-gaugeyu-gauge?orgId=0

官方更多配置： https://play.grafana.org/d/KIhkVD6Gk/4-gauges?orgId=1&refresh=10s

**基本配置（Panel Options）**：标题、描述、Panel透明度、Panel的外链配置

**数据配置（Value options）**：值展示方式、最大返回行数、展示字段

- Show: 
    - All Values，表示在选定的时间范围内显示所有的采样值，如语句中的 index, count(1) 的值
    - Calculate，需要根据下方的Cal( Last, First,Min,Max等）中的具体选项来展示一个单一的数据，如，将只显示count(1)的值
- Limit: 显示值的最大上限
- Fields: 选择Numeric Fields，表示从计算结果中 显示数值的value

**图形配置（Gauge）**：

- Orientation:  水平 或者 垂直显示，默认水平
- Show threshold labels： 展示thesholds配置的阈值基线，以阈值（数值）展示在最外层
- Show threshold markers：展示根据thesholds渲染的 阈值颜色标记, 再最外层显示

**文字大小（Text size）**：Title Size，Value size

**标准配置（Standard options）**：单位、最大/小值设置、小数点位数、指标展示名、颜色方案

- Color schema**:** 图表显示的样式与颜色, 在gauge中，通常选择 From thresholds(by value)， 因为需要根据数值展示不同的颜色和标记

**数据链接（Data Links）**： 数值点上点击，可以外链；参考： [Data Links](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235311689#id-03【BATGrafana】-3DataLinks)

**阈值配置（Thresholds）**：阈值类型、阈值线显示

1. **Base**代表从起始值开始。
2. 如图含义为：从起始值至70数值时为绿色，从70至85为橙色，大于85为红色
3. Thresholds mode： 表示数值显示为绝对值、百分比

### PieChart Panel

Grafana原生插件，在饼图上不适合展示非常多的数据，即使是用多种颜色区分在视觉上也不如条状图 

官方Demo: https://play.grafana.org/d/ktMs4D6Mk/5-bar-charts-and-pie-charts?orgId=1

**基本配置（Panel Options）**：标题、描述、Panel透明度、Panel的外链配置

**数据配置（Value options）**：值展示方式、 展示字段

- Show：通常为【Calculate】
    - Calculate（计算）: 选择通过计算的方式 ，按照时间，将数据聚合展示
    - All Values（所有值）: 在指定查询时间范围内，显示Pie 的所有值，会包含time 和 number
- Calculation(计算)：计算方式，Min（最小值）| Max（最大值）| Mean（平均值）| Total（总计）| Last*（当前非空的值）| Last（当前值[含空值]）
- Fields: 值使用什么字段来展示，通常是Numeric Fields, 会自动获取 查询到的数值

**饼图配置（Pie Chart）**：饼图样式和图上展示的内容

- Piechart type ：饼图展示的类型，有Pie（饼图） 和 Donut（甜甜圈）
- Labels ：     显示在饼图上的内容

**图例配置(Legend)：**显示为List 还是 table格式, 位置在底部还是 右侧, 图例展示的数据类型是百分比还是普通数值

**鼠标悬浮（Tooltip）**： 悬浮显示所有线｜单条线、线的排序

**标准配置（Standard options）**：单位、最大/小值设置、小数点位数、指标展示名、颜色方案; 参考 [StandardOptions](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235311689#id-02【BATGrafana】-2StandardOptionsStandardOptions)

**数据链接（Data Links）**： 数值点上点击，可以外链;  参考 [ Data Links](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235311689#id-02【BATGrafana】-3DataLinks)

**阈值配置（Thresholds）**：阈值类型、阈值线显示

### Statusmap Panel

Statusmap panel可以显示多个目标**随时间**的推移所产生的离散状态。

在Visualization中选择【Statusmap】 即可进行Statusmap可视化的一系列配置。

**Colors**

* Color Mode = opacity 不透明度模式
    * Scale 渐变程度
        * 不透明程度可选 ：sqrt 开方 | linear 线性
        * 当选择sqrt开方时则可选Exponent开方指数
* Color Mode = spectrum 光谱模式
* Color Mode = discrete 离散模式：离散模式允许用户自定义value等于具体某个值时的展现颜色。

**Color scale **

* Min value 颜色展现范围的最小value

* Max value 颜色展现范围的最大value

*如设置http状态码 可以设置为0~600，这样200会是绿色，400+会是红色*

**Display**

show legend：是否展示图例

Show tooltip：是否崭新啊鼠标悬浮图层

Y axis sort：Y轴排序逻辑

**Bucket**

Multiple values：多值时颜色由显示模式（Colors）决定

Display nulls 空值处理方式：as empty 置为空 | as zero 当做0值显示

Min width：单元格最小宽度

V spacing：垂直间距

H spacing：水平间距

Rounding：每个单元格的圆角配置

### Flowcharting Panel

详细介绍：http://conf.ctripcorp.com/display/FRAM/15+Flowcharting+Panel

FlowCharting是一款Grafana的插件，这个插件是基于[http://draw.io](https://link.zhihu.com/?target=http%3A//draw.io)在线绘图库的，用[http://draw.io](https://link.zhihu.com/?target=http%3A//draw.io)可以完成非常复杂的图表的绘制，比如网络拓扑图、数据流量图、技术架构图等，然后我们可以用FlowCharting为图表中的各个元素绑定动态的数据，从而完成一张动态的图表。[ 案例参考](http://hickwall.ctripcorp.com/grafanav2/d/F0yubq_7z/nocce-shi-fa-bu-bian-dong-kan-ban?refresh=30s)

如您在使用此插件 制作业务大盘时有问题，请进入 **[监控可视化技术支持群 ](https://c.ctrip.cn/apk/cchatcp.html?action=openChat&type=groupchat&jid=2047168584248983701)**

Grafana-Flowcharting 相关资料: https://grafana.com/grafana/plugins/agenty-flowcharting-panel/

#### 绘制图形 -Visualization

**FlowChart**：可以直接在 Grafana中借助 [http://draw.io](https://link.zhihu.com/?target=http%3A//draw.io) 来绘制图形, 点击【Edit Draw】进行修改，绘制完毕save之后，会自动将图形代码填充。

- Edit Draw：在线编辑当前图形。
- Prettify：格式化图形代码。
- Minify：合并图形代码。
- Compress/Encode：加密图形代码。
- Extract/Decode：解码图形代码。

生成好绘制的图形后，我们就可以开始调整看板的基础配置。

**Advanced**：

- Bg Color: 可以调整 grafana 看板背景色。
- Scale: 图形填充大小设置
- **GetAllStatusAlert** ： 此处跟下方 3 中的*【Thresholds 】 对接告警中台展示* 有关，此开关不打开的情况，若关联了ruleid, 仅展 正在发生（未解决未关闭）的告警，打开之后，可以看到时间段内，所有状态的告警事件。 适用于 复盘场景

#### 为图形绑定监控数据源 - Queries

我们需要为关键图形元素添加监控数据源。根据需求可以在报表中添加相应的 Query 语句

切换到【Queries】中配置查询语法，*此处各个查询语句的"legend"，会用于下文中关联图形使用*

如果您不想要设定查询语句，但控制拓扑图中严重程度展示，请将

- Queries To选择成【-Grafana-】
- 下方的查询语句设置为random walk随机数

*解释：flowcharting中必须要包含一个查询语句，才能保证下方的mapping rule的颜色展示。 所以就算您完全希望用 绑定ruleid的形式展示，也需要设置虚拟(假)的查询语句。*

再 参考3中的【Thresholds 】 对接告警中台展示。

#### 配置图形展示规则Mapping Rule、关联告警规则- Visualization

在报表中可以通过添加 Rule ，来设置我们图形的展示效果。 切换到【Visualization】中，鼠标拖动到下方【Mapping】部分，添加rule

如上图所示

**Options** 是用来关联上一步中添加的监控数据源。

- Rule name: 规则名称。 可以使用legend Name
- *Apply to metrics*： 与哪个查询语句关联 (支持正则), 鼠标点击之后，会自动下拉出查询语句的名称【legend Name】。 
- Aggregation: 为数据源选择一个聚合。

**Type** 是用来设置数据源展示的指定单位和类型。

- Type：类型，Number/String/Date, 此选择决定了下方的Thresholds设置的方式。
- Unit：单位。
- Decimals：保留几位小数。

**Thresholds**(OK>Warn>Critical) ：阈值配置, 用来设置图形展示的颜色变化规则, 类似于手动配置阈值区间，以达到告警的效果

有两种方式支持： 1 是根据查询语句的结果手动设置某个区间 。2 是对接告警中台，关联告警规则

方式一：根据查询语句的结果手动设置某个区间

- Thresholds: 用来设置颜色变化的方式
    - 若上方的Type值为Number： 则**设置区间数值, 比如 150,200** 即表示 <150 对应 OK; 150<x<200 对应Warn; >200 对应Critical。下方的图1
    - 若上方的Type值为String：则设置语句返回的string值，来判断严重程度。 下方图2

- Color Mode: 颜色显示模式, 默认fill, 即根据区间数值反应的颜色进行填充
- Colors: 分别对应Thresholds（OK>Warn>Critical）体现的三种颜色
- Invert：倒序排列（Critical > Warn > OK）。
- Icon state：icon图标展示, 若选中, 则在当前图形为Ctritical时, 右侧会出现警报的闪烁图标

方式二：对接告警中台，关联告警规则

展开Use OneAlertRule，即可选择告警中台的规则, 保存后，将根据当前看板选择的时间范围去获取正在发生的告警事件，一旦有正在发生的告警事件，图中的颜色将会设置为Ctritical对应的色块

- Bind Alert Rule: 绑定告警规则id
- Bind Instance： 绑定实例, 输入内容后，光标移至空白处，自动填充值
- Bind Level： 绑定告警等级, P0~ P4

以上三个内容可任意组合，请按照实际需要进行设置。 注： 根据以上组合获取到的告警事件，每个规则id 仅获取最新的一条进行展示。

**注: 若同时设置了两种方式，两者任意一个满足条件，都将把严重程度设置为Critical且显示其对应的颜色**

**Display**

Color on：默认Always，影响Color Mapping

Value on：默认When Mertic display，影响Text Mapping

**Tooltips** 开启后，将鼠标移动到具体图形上，会自动展示对应监控值一个周期状态， toolTips展开之后，可设置 4.1 Graph ToolTips 和 4.2 OneAlert Tooltips

- Label：弹出窗口中展示的标签（默认和查询语句命名的"名称"相同）。
- Tooltip when state is : Tooltips什么时候展示。 Always | Warning/Critical
- Tooltip Direction：垂直显示还是水平显示。
- Color with state：弹出的图形线条颜色是否和服务状态颜色保持一致。

**Graph Tooltips** 用来设置弹出图形展示的效果。

- Graph type：使用线性或者柱状图。
- Graph Size：弹出的图形大小。

**Link**：Object as link打开，以下的配置会影响link Mapping

**OneAlert Tooltips** ： 用于当您关联了告警规则，则将当前时间范围内正在告警的事件 列出展示，并可点击外链。 **如果您关联了告警中台的ruleid，建议您开启此选项；**

#### 关联具体图形以展示颜色 或者 文本

**Color Mappings**：关联具体图形以展示颜色 或者 文本

将具体的图形关联到配置好的 Rule 中，这样就可以在 Grafana 看板中看到具体图形的状态颜色了

id：即每个图形的 id, 图形id的选择 请点击“超链接”按钮，然后根据提示，点击您关联的图，会自动将id填充进去，如下动图

- 叉叉图标 : Remove line/shapes from current rule
- 眼睛图标: Hide and ignore line from current rule
- 回形针图标: Click on it to select shapes/object in panel and auto fill input with id object.

### Overrides

#### Multiple y-axes

多Y轴可以 当一个面板Y轴上需要展示不同的单位和维度数据时;  需要使用 Overrides

配置如下图（从左至右依次进行） tips： Add override property之后，可以搜索对应的属性名称，如图4

选择属性名

* Fields with name：直接设置指标名
* Fields with name matching regex：正则匹配指标名
* Fields with Type： 根据返回的数值类型，只包含了Time 和 Number类型
* Fields returned by query：查询编号 A、B、C

Add override property：增加属性

* Axis  Placement：选择Right，可以看到左右双Y轴
* Standard optionsUnit：可以选择Y轴的单位

#### Negative Y

Negative Y是指：将值显示为负数， 并将数据展示在负Y轴, 如通常网卡进出流量可以展示在正负Y轴上；

1、切到【Overrides】Tab, 点击 + Add field overrides

2、可以选择, 可以根据自己需要去选择 获取的字段，这里以网络进的流量为例，选择了【Fields with name】

3、点击 + Add override property, 在弹出的框中输入“transform” 搜索, 选择 Graph styles > Transform ， 选择【Negative Y】 

#### Fill below to

将两者值充满颜色 ，由上往下从大值到小值

### Transform

在将查询结果渲染给 统计图表进行格式化展示前，您可以通过Transform（数据转换）对查询分析结果进行二次处理。例如合并多个查询分析结果、对查询分析结果进行分类等。

https://grafana.com/docs/grafana/v9.0/panels/transform-data/transformation-functions/

#### Reduce

把一个运算应用到每个字段，返回一个标量值。时间域会被删除。且是针对多个请求而言

生成新的表格，字段名为行（除Time外），对字段的数据进行max、min等聚合操作, 结果为列（每种聚合一列）； 可将多个查询的结果根据相同Time进行合并；

选项说明：

| 选项             | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| Mode             | Series to rows：每个字段对应一行；针对运算，每个字段都有一个结果。多个计算，则每个字段有多个结果(结果是一张表)。<br />Reduce fields：保持当前结构，但将每个字段折叠为单个值, 结果类似前一种，但连个请求处理后，对应的是两个表。 |
| Calculations     | 针对字段的运算(时间，值)                                     |
| Include          | 选择Reduce fields才有这个选项。即是否包含时间域。            |
| Labels to fields | 选择Series to rows才有这个选项。                             |

举例一：选择Series to rows模式， 并使用First 和 Last的计算方式

```perl
#原始数据
Query A：
Time				Temp	Uptime
2020-07-07 11:34:20		12.3	256122
2020-07-07 11:24:20		15.4	1230233

Query B：
Time				AOI		Errors
2020-07-07 11:34:20		6.5		15
2020-07-07 11:24:20		13.2	5

#转换后
Field		First		Last
Temp		12.3	15.4
Uptime		256122	1230233
AQI			6.5		3.2
Errors		15		5
```

举例二： 选择Reduce fields模式， 并使用Last的计算方式：左侧是原始数据 - 》 右侧转换后的数据

```perl
#原始数据
Query A：
Time				Temp	Uptime
2020-07-07 11:34:20		12.3	256122
2020-07-07 11:24:20		15.4	1230233

Query B：
Time				AOI		Errors
2020-07-07 11:34:20		6.5		15
2020-07-07 11:24:20		3.2	5

#转换后
Query A：
Temp	Uptime
15.4	1230233

Query B：
AOI		Errors
3.2	5
```

*注意：聚合时会删除时间字段,  使用此函数转换后, 最佳适配的插件是 Table Panel*

#### Merge

类似sql中的join，将多个查询的结果组合成一个结果，目的是合并不同的字段数据；如果共享字段包含相同的数据，则值是可合并的。合并后的内容 适用于 Table Panel插件

#### Outer Join

使用此转换按字段连接来自结果集的多个时间序列(合并请求很有用:请求源不同，请求的返回的数据结构相同)。

#### Concatenate fields

连接字段。将多个字段合并成一个。

#### Convert field type

转换字段类型

#### Filter data by value

这种转换允许你直接过滤你数据，然后从请求结果种删除掉某些数据。

对于你定义的一个或多个条件，若请求的数据匹配上了，你可以选择包含它，或者排除它。条件是作用于选定的字段(field)的。

- 对于任何字段(field),条件可以是：
    - 正则表达式
    - Is Null：值为null则匹配上
    - Is Not Null：值非空则匹配上
    - Equal：若值等于指定的值，则匹配
    - Different：和Equal相反
- 对于字段是数字的，条件可以是：
    - Greater:值大于指定的值，则匹配上。
    - Lower: 值小于指定的值，则匹配上。
    - Greater or equal: 值大于或等于指定的值，则匹配上。
    - Lower or equal: 值小于或等于指定的值，则匹配上。
    - Range: 一个范围。指定最大值，最小值，包含这两个值(闭区间)。

选项框如下:点击"Add condition",即可出现Field所在行(可点击多次，生成多个条件)。

| 选项        | 说明                                                         |
| :---------- | :----------------------------------------------------------- |
| Filter type | 含Include(使图像***只包含***满足条件的数据)和Exclude(使图像排除满足条件的数据)两个类型。若选项为亮白色，则表示被选中。 |
| Conditions  | 针对多个条件而言，***Match all***(\==and)要匹配所以条件；***Match any***(==or)匹配以下条件 |
| Field       | 依次填写要操作的字段，条件，指定的值(用以比较)。             |

#### Series to rows

***合并几个时间序列为一个。结果含三列：时间，字段名字(metric), 值。***

将两个query中的不同字段转换到数据表格中，适用于每个query只有Time和值两列的情况， 转换之后，适用于 Table Panel插件

```perl
#原始数据
Query A:
Time				Temperature
2020-07-07 11:34:20		25
2020-07-07 10:31:22		22
2020-07-07 09:30:05		19

Query B:
Time				Humidity
2020-07-07 11:34:20		24
2020-07-07 10:32:20		29
2020-07-07 09:30:57		33

#转换后数据
Time					Metric			Value
2020-07-07 11:34:20		Temperature 		 25
2020-07-07 11:34:20		Humidity			24
2020-07-07 10:32:20		Humidity			29
2020-07-07 10:31:22		Temperature			22
2020-07-07 09:30:57		Humidity			33
2020-07-07 09:30:05		Temperature			19
```

#### Organize fields

使用此函数来重命名、重新排序或隐藏查询返回的字段。

注：此函数仅适用于具有单个查询的面板。如果您的面板有多个查询，那么您必须先其他函数或删除额外的查询。

选择Organize fields将默认显示所有可设置的字段， 以下Field, Last* Mean 均是字段默认名

#### Sort by

此转换将每个frame按照配置字段排序，若检测到`reverse`，则值会逆序返回。

#### Add field from calculation

使用两个其他的字段(field),或者两个query的结果进行计算，获取一个新的字段。

|                    |                                                              |
| :----------------- | :----------------------------------------------------------- |
| Mode               | 模式选择。有Reduce row和Binary option两个选择。<br />Reduce row：将选择的运算方法分别作于于两个filed(或者请求结果)。<br />Binary option：对来自两个选定字段(或者两个请求的结果)的单行中的值应用基本数学运算（求和、乘法等)。注意：Reduce row本质是对一个字段(请求)的处理。 |
| Field name         | 要进行计算的字段(field),这两个字段可以来自两个请求。         |
| Calculation        | 若选择**Reduce row**，则点击下拉框即可看到可以使用的运算。<br />若选择**Binary option**，则可以对选中的两个字段进行基本四则运算(±*/) |
| Alias              | 设置新的字段的名字(可选)。直接设置值作为legend的名字，可以用数据点的tag作为表的名字。语法[[tag_tagName]]。详见https://grafana.com/docs/grafana/v7.5/datasources/influxdb/ |
| Replace all fields | 隐藏用于计算的两个字段，显示新生成的字段(针对图像而言)       |

### 快捷键

Grafana 有许多可用的快捷键。按键盘上的 ? 或 h 键，可显示您的 Grafana 版本中可用的所有快捷键。

- Ctrl+S: 保存当前仪表盘。
- f: 打开仪表盘搜索器/搜索。
- d+k: 切换信息亭模式（隐藏菜单）。
- d+s: 仪表板设置。
- Ctrl+K: 打开命令调板。
- Esc: 在全屏视图或编辑模式下退出面板。还可以从仪表盘设置返回仪表盘。

将鼠标悬停在面板上，可以使用一些针对该面板的快捷键。

- e: 切换面板编辑视图
- v: 切换面板全屏视图
- ps: 打开面板共享模式
- pd: 复制面板
- pr: 移除面板
- pl: 切换面板图例

# 指标

## 监控数据

数据源指的是 ，系统中 支持 来自哪些地方的数据，进行配置监控, Grafana中目前支持 时序类型（Prometheues，OpenTSDB, InfluxDB, Victorial Metrics） ，Logging/Document类型的（ElasticSearch） , 以及SQL( MySQL, ClickHouse） 等多种类型。 最常用的是Victorial Metrics 和 ClickHouse

由于信安的安全性要求，目前暂时不接Mysql类型的 数据源. 

集团目前使用埋点, 都是将指标写入了Victorial Metrics（以下简称VM）, 完全兼容Prometheus的查询语法promQL

- FX-Dashboard:   soa.xxx,slb.xxx,qmq.xxx,credis.xxx 等框架埋点在FX-DASHBOARD
- APM-Dashboard: 其他的用户自埋点类似 ibu.xxx,htl.xxx,flt.xxx在APM-DASHBOARD
- SRV- 是基础服务相关指标，如数据库、ES、HDFS、Kafka
- CIS- 是基础设施相关指标，如系统CPU、MEN，网络、JVM

## 监控指标解释

公共的指标 CIS-SYSTEM

| 指标类别 | 名字                          | 说明                                                         | 用法案例                                            |
| :------- | :---------------------------- | :----------------------------------------------------------- | :-------------------------------------------------- |
| cpu      | sys.cpu.load1.per_core        | 系统指标`sys.cpu.load1`除以cpu核数（系统指标`sys.cpu.number`）所得值； `sys.cpu.load1`： 系统负载，任务队列的平均长度，1分钟到现在的平均值 | sys.cpu.load1.per_core_value{endpoint=~"$hostname"} |
|          | sys.cpu.util_percent          | cpu使用率，计算方式：100 - `sys.cpu.idle_percent`（空闲cpu百分比） |                                                     |
|          | `sys.cpu.user_percent`        | 用户空间占用cpu百分比                                        |                                                     |
|          | sys.cpu.system_percent        | 系统内核空间占用cpu百分比                                    |                                                     |
|          | sys.cpu.iowait_percent        | 一分钟内有百分之几的时间属于以下情况：CPU空闲时间、并且有仍未完成的io请求 |                                                     |
|          | sys.cpu.util_percent.per_core |                                                              | Docker容器                                          |
| Memory   | sys.mem.available             | 机器可用内存                                                 |                                                     |
|          | sys.mem.total                 | 机器总的物理内存                                             |                                                     |
|          | sys.mem.available_percent     | 机器可用内存百分比， sys.mem.available_percent = sys.mem.available / sys.mem.total |                                                     |
|          | vm.totalavaillable.size       | vm.totalavaillable.size = (sys.mem.available + balloon (balloon 此项如果没有，则为0) ) / sys.mem.total |                                                     |
|          | sys.mem.used.rss_value        | 进程常驻内存； RSS是Resident Set Size（常驻内存大小）的缩写，用于表示进程使用了多少内存（RAM中的物理内存），RSS不包含已经被换出的内存。RSS包含了它所链接的动态库并且被加载到物理内存中的内存。RSS还包含栈内存和堆内存。由于部分内存是共享的，被多个进程使用，所以如果将所有进程的RSS值加起来可能会大于系统的内存总量。申请过的内存如果程序没有实际使用，则可能不显示在RSS里 |                                                     |

# 告警

告警的级别设置和响应时间要求

| 报警级别       | 影响范围                         | 建议通知方式                                      |
| -------------- | -------------------------------- | ------------------------------------------------- |
| P0（Disaster） | 整个BU，仅供事件升级用           | 该BU所有oncall和所有团队主管                      |
| P1（Critical） | 关键功能无法工作（有覆盖率要求） | 24X7多渠道实时通知oncall或owner，统计反应时间     |
| P2（High）     | 关键功能受影响（有覆盖率要求）   | 工作时间多渠道实时通知oncall或owner，统计反应时间 |
| P3（Moderate） | 单机或一般功能受影响             | 原生通知方式                                      |
| P4（Low）      | 基本无影响，数据统计用           | 原生通知方式                                      |

集团的告警数据 均已入中台,  中台涵盖了 【策略配置】【通知配置】【告警事件处理】 三个重要的组成部分。

- 策略配置： 指的是告警满足什么条件才发生。 其中策略就包含[阈值策略](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1246111543)，与 [智能策略](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1243669050)。
- 通知配置： 指的是告警发生了，该通知什么人处理
- 告警事件处理：指的是告警真实发生了，有一个统一可以看到所有告警，并进行 响应与解决的操作



其中【策略配置】即告警规则，集中在HIPD, Hickwall, SiteMon三个主流的告警系统中配置：

- HIPD： 也称之为Hickwall Portal，一款基于告警中台而产生的产品，主要用于数据配置和搜索， 告警的所有(包含Hickwall,Sitemon) 【策略配置】/ 【通知配置】都可在它上查到。 - 研发常用
- Hickwall: 集团应用/系统相关的 可视化指标监控系统，在上面除了可配置 可视化仪表盘，也可配置告警，数据终将全部 同步显示在 HIPD - 研发常用
- Sitemon: 集团业务订单(机票, 酒店等等) 的监控系统，可配置告警,数据终将全部 同步显示在 HIPD   - 业务常用

### 阈值告警

BAT中配置了可视化图表后, 可以很方便的进行 告警规则设置。

注：prometheus的数据源, 不支持针对带参数的查询语句进行 告警

阈值：即一个划分正常与异常的界限，可以是一个确定的点，也可以是一个范围内，当采集值超过了这个界限，既把这个采集点作为异常点， 异常就会就产生告警事件，通知对应的owner。

#### Meta

- BU： 告警所属Bu， 必填
- ProductLine**：** 告警所属产品线
- AppIds： 告警关联的AppIds， 必填
- NameSpace ： 命名空间，可自行定义
- CustomTag： 自定义tag，最大可自定义5个

#### Rule

- Name： 告警规则名称，Name支持hickwall中设置的 [Variables](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1235306296) 
- Evaluate every: 执行频率, 1m表示每分钟检查一次数据，单位格式为 m
- For：连续xx m/h/d （例：5m 即表示连续满足条件5分钟）检查结果都是异常则告警， 如果 *需要满足条件 立即告警，则将 For的输入框 写成 0（注意：不带任何单位的0）*

#### Conditions

* WHEN 

    avg()：对查询语句得出的结果进行聚合，eg: avg是对获取到的每条曲线的数据求平均值

    | 运算函数         | 说明                                                         |
    | ---------------- | ------------------------------------------------------------ |
    | avg              | 平均值，相当于sum/count                                      |
    | count            | 数据点数（在单位时间里去抓取了几次metric），一般很少用，例如，配置了每隔15s去抓取1次数据，在1分钟内的count就是5，5分钟内的count就是21 |
    | count_not_null   | value不为空的count                                           |
    | diff             | 起始值和最终值之间的差异：`newest - oldest`                  |
    | diff_abs         | 起始值和最终值之间的差异绝对值：abs(`newest - oldest)`       |
    | last             | 最后一个值                                                   |
    | max              | 最大值                                                       |
    | min              | 最小值                                                       |
    | percent_diff     | `(newest - oldest) / oldest * 100`                           |
    | percent_diff_abs | `abs((newest - oldest) / oldest * 100)`                      |
    | sum              | 这个metric获取的所有value的总和，例,配置了每隔15s去抓取1次数据，在1分钟内的抓取到的这个metric对应value分别是3,4,5,4,2，那么1分钟内的sum就是3+4+5+4+2 |

* OF

    query(A, 5m, now)：A:查询表示式的名称，5m,now:时间区间，表示5分钟前到现在。eg: 查询5分钟前到现在表达式A的数据

* RingCompare：环比告警开关，打开时,，将可以设置环比天数，查询时间区间等信息

    - OffsetDays: 环比几天前。 
    - Interval: 与②中的5m语义相同, interval前 ~ 现在的数据。 举例说明: 环比7天前, 5分钟前到现在的数据 。
    - 比较方式： 上升%、下降%、波动%（上升或者下降）

* IS ABOVE

    比较方式：大于/小于在某区间/不在某区间等方式。注：打开RingCompare,此项不可见

* Level：告警等级

* AlertTime：告警触发时间，可选0:00 ~ 24:00

* AlertDate：告警触发日志，可选周一 ~ 周日

* Default：当前告警触发时的通知方式，默认Default，此部分将在【Notifications】中解释

#### Test Rule

配置好告警规则之后， 单击右侧上方`Test Rule`，测试告警规则,看结果是否符合预期; 

- OK表示没有触发规则
- Pending表示触发规则,  但是需要达到配置条件中 持续时间，如，Evaluate every 1m For 5m ， 就表示必须持续5分钟pending, 才会变成Alerting

注： 如果不需要持续多少分钟才告警，检测到的数据点超过阈值即告警，可把For 5m 设置成 For 0

- Alerting 表示告警真正触发
- nodata表示没有查询到数据，空字符串表示查询有问题(例如后台挂了)

#### No data and error handling

| SET STATE TO    | 效果                                                         |
| :-------------- | :----------------------------------------------------------- |
| OK              | 无数据或者数据值为null时，都不会触发告警                     |
| Keep Last State | 与最近一次的告警状态保持一致                                 |
| No Data         | 无数据或者数据值为null时，触发告警，通知title后面有No Data文字 |
| Alerting        | 无数据或者数据值为null时，触发告警，通知title后面有Alerting文字 |

#### Notifications

Default：默认设置，即使用告警中台中定义的各个级别的通知方式进行通知，不可删除，必填

自定义：用户根据级别自定义的通知，例如P0级别的自定义通知，同级别的自定义通知会覆盖掉默认通知

* Level：可以多选，一个级别只能自定义一种通知，邮件标题和短信内容上会加上这个信息，与上面【Conditions】中的规则 Level 匹配
* Send to：通知方式，其中onealert为默认的，即发送到告警中台，另外还支持短信、电话(TTS)、邮件、TripPal
* 通知人配置：以下必须至少有一个可以找到通知人。否则通知配置无效。OwnerAdmin和上三个都配置的话，则都会通知
    * Users：需要通知的人，一般不配置邮件组，*每天早上8:00同步*
    * Mail Group：邮件组
    * Oncall：排班表，既可以打电话给值班人员，也可以发邮件和短信给邮件组, 在 oncall/ 中配置排班后, 在此可搜索到
    * OwnerAdmin：是否发送到appid的owner和admin（需要在meta中填写appid）

- Message：用户收到的通知内容