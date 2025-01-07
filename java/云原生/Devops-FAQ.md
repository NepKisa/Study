# skill

```perl
git config --global core.longpaths true
```

# 日志

## pod

出现Insufficient cpu的日志，reset后可以恢复

> ```
> 2024-02-29 17:10:32` - `Pod|r100013446-91702329-gcwgt` - `Failed, Failed to pull image "gitlab-ci/100013446:master-f7fc52e0-20240229153653": rpc error: code = Unknown desc = Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
> 2024-02-29 17:10:32` - `Pod|r100013446-91702329-gcwgt` - `Failed, Error: ErrImagePull
> 2024-02-29 17:10:33` - `Pod|r100013446-91702329-gcwgt` - `SandboxChanged, Pod sandbox changed, it will be killed and re-created.
> 2024-02-29 17:10:35` - `Pod|r100013446-91702329-gcwgt` - `BackOff, Back-off pulling image "gitlab-ci/100013446:master-f7fc52e0-20240229153653"
> 2024-02-29 17:10:35` - `Pod|r100013446-91702329-gcwgt` - `Failed, Error: ImagePullBackOff
> ```

## 事件中心

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

* IP实例消失，但在SOA平台上存在，SOA平台数据异常

* 两个appid创建同一个域名的入口需要path不一样

* 域名无法搜索到appid可能是虫洞域名

* HPA有多个指标，有一个满足就会扩容，缩容时需要全部满足才会缩容，若QPM无数据则无法判断

* 流水线权限显示没有owner或maintainer时（git上已配置）可以在captain的流水线列表同步成员权限

* 机器反复重启，查看下内存CPU指标，可能是机器配置低导致的，sys.cpu.throttled_time是CPU资源不足，调高配置解决

* 更新页面的事件中心是全局的，无论更新的是啥，显示的都是全部的日志，更新hpa与pod无关

* 实例磁盘满了不会自动重启，只能手动进机器清理磁盘恢复或者reset

* JVM参数不设置HeapDumpBeforeFullGC和HeapDumpAfterFullGC是因为太频繁会把磁盘撑满，此参数是在FullGC前后dump

* 点火未进行且bat无任何日志时可以使用arthas，thread -b查看阻塞的线程

* 8.1.0bom的qconfig会线程阻塞导致发布卡住

* `Module build failed: TypeError: Invalid PostCSS Plugin found`这个报错网上有issue

* Node的lint太慢，可以只检查必须的模块，非必要的都配置eslintignore，.eslintigore google一下

* 堡垒机发布卡住：可以新建变更重新发布尝试

* 临时关闭UT覆盖率检测：将应用降级为非核心，再删除红线即可

* 曾经发布成功的版本再次发送未必成功，外部组件可能有变化导致发布失败，如qconfig、数据库等

* npm deploy时缓存问题可以修改 "prepublishOnly": "rm -rf node_modules/ package-lock.json && npm install && npm run build"

* 删除fork的原项目可以先在general取消fork【owner权限】防止关联删除

* SCM new Chinese检查可以在isssue里搜索tag【new Chinese】查看，general里可以看min chinese code

* SCM approval没权限会报错【An error occurred while submitting your approval】，approval by 后面的是需要点的人【还没点】

* SCM deploy失败但maven仓库有【偶然现象。可以寻找二线更改gitlab状态】

* Captain 固定IP，==镜像tag不变的情况下会优先使用本地的，务必更新tag==

* 资源不足可以reset释放资源

* 发布系统的堡垒机不接流量，ContainerNotReady不接流量【70s缓冲】，拉入堡垒会接流量【金丝雀】

* 切换root用户sudo -i失败可以用sudo -Es

* 关闭HPA无法扩缩容，删除HPA可以

* 堡垒发布卡住，要所有实例ready才能成功，可以reset

* 新建变更过程卡住【如卡在滚动发布】，需要所有实例ready才可进行下一步

* 流水线执行请求体过长报错413【find . -name *.pyc -delete】，包太大了，用自定义 job，把 package 和 image 合成一个

* JAVA17 ZGC会导致启动时占用内存极大，设置Xmx与Xms相等即可

* this commit is not fast forward to target branch 配置了自动合并，rebase或者merge

* 一个应用可以最多创建的fat环境数量没有限制

* No asset id or system user id found，申请登录权限

* sonar问题，能修就修 不能修就忽略

* tomcat7和jdk17的兼容问题导致点火404，服务启动访问不到vi/health

* 发布规范可以修改DR等待时间

* sonar 没有办法去掉，jacoco 可以去掉。机票的规则是所有应用要进行code review，如果用户需要去掉红线，发送邮件到部门pmo，抄送scm团队。pmo 同意了，二线可以加白。

* pages服务访问页面404 ，要在public下【pages 部署必须把 index 放 public  里】。可以找二线清下缓存

* deploy一直转圈，可以换版本重发，或者让二线改成失败重发

* 查看sonar报告接口：merge_requests/468/pipeline_reports?type=sonar&detail=incr

* 出现中文红线时可以考虑申请非国际化

* gitlab token 新版本只能用个人 token，账号密码不行了

* Node UT失败，settings > ci/cd > general pipeline > Use separate caches for protected branches 勾掉，模板有问题【cache的key不一样】，自定义test，生成的 clover 报告里，覆盖率是0，jest 测试不用 nyc

* This merge request is not associated with any iDev issues. 重新绑定下idev需求和分支。建议需求发布后不要继续在同一个分支上开发，最好是建fix分支修改。绑定分支发布后，gitlab 这边会判断 idev 需求已发布，所以不会再展示

* NodeJS Serverless没有红线配置

* go 服务编译 下载组件失败404，改 golang variables ，使用新的 artifactory 服务代理 golang 包，不用 goproxy.release  了，就是要配置一遍 公共账户 和 token 到你仓库 settings variables 里

* gitlab ci Uploading artifacts as "archive" 413，自定义job，合并成一个job

* sonar规则不是SCM制定的。误报只能行忽略，或者项目忽略

* UT new覆盖率接口：/merge_requests/28/pipeline_reports?type=jacoco&detail=incr

* new 覆盖率changes diff 过多会有部分统计不到，这种只有两种方法，补全能看到的，或紧急合并， test job 跑完后，可以访问/merge_requests/856/pipeline_reports.json?detail=incr&type=jacoco查看详情

* arm和amd镜像的sha256不一样，打镜像时用默认的基础镜像才是多架构的

* mac require路径mac大小写不敏感、Linux敏感

* maven deploy 建议直接升到 1.0.14版本，不然之后部署每次都要找二线改状态

* 批量删除分支Api /help/api/branches.md#delete-repository-branch

* 非标准镜像，登录失败ssh: handshake failed: ssh: unable to authenticate，attempted methods[none publickey]

* sonar不支持21的语法且目前没有升级的打算，会支持 java21 的扫描，但是不会支持这些非常新的参数

* pass页面有未删除的group、但销毁group按钮是灰色，显示captain删除，可以删除disabled，再点击销毁

* 点火404，应用没起来，SpringBoot3选9就是这样，它只能用10，SpringBoot 1和2选10也是同理

* 自定义tomcat参数：【java 规范库，和 extraenv.sh 写在一起的 自定义 tomcat server.xml 的方式。和 extraenv.sh 放在一个目录下，就是需要从 容器里先找个 server.xml 完整 copy 下，再修改自定义配置的部分】

* cpuset开启后重新发布才会生效，可查看yaml文件确认resource.limit和resource.request

* 查看 catalina.out 日志可以找到 Spring Boot 的版本。如果是 2 开头的，出现类似的 javax 开头的类 ClassNotFound，并且用户用的是 Tomcat 7，那就让他们升级到 Tomcat 8 试试

* 现在公有云机器是弹性的 可能会回收 影响固定IP的质量 需要固定IP的话建议注册vm应用

* 令牌检验不通过，itoken/ 测试下令牌，尝试重新绑定

* Java 11 里 ZGC 还不支持 linux/arm64，15以后支持

* git 邮箱检测未生效，回退一步重新提交生成 git commit

* HPA 应用是多集群架构，在xy-b 有2个， xy-c有一个 实例，两个集群的hpa是独立的【指标除2计算，xy-b 这边，平均是大于20的，所以缩不到1】。监控页面，左上k8s_cluster 切集群，挨个看xy-a, xy-b, xy-c 是否有实例

* captain开通流水线400，仓库不存在，由于仓库改名，可以从git开

* 合并到master触发，那只设置ref master 就行了，master 分支上代码有变更就会触发

* only one file can be sent as raw，artifact里clover只能上传一个文件

* nodejs SLB访问502，健康检测不通过，健康检测地址修改为/slbhealthcheck.html

* idev QA测试和镜像发布者不能是同一个人

* captain war包链接7天就过期

* paas机器数量最多三台 全局限制

* 同步入口CR卡住，可以点暂停再点继续（尝试让 syncer 重新同步一遍数据），或者重新发布

* paas显示发布中，但是发布记录找不到，状态没同步回来，用相同版本覆盖发布一次这个group

* it Helpdesk 申请 SCM 公共账号

* MR的时候，同commit结果会覆盖，可以重新collect刷新

* sonar的中文检测不检测注释，修改等同于新增，nosonar可以对中文检测生效

* paas发布失败无日志，webinfo重启服务器，十分钟后重新发布

* MR没有revert按钮，fast-forward 没有生成merge commit,没有办法直接revert 整个mr，只能一条一条revert

* maven deploy是使用的上一次成功的配置，更改settings后多试几次

* Java类型的容器可以通过指定基础镜像来实现tomcat

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

* 有状态应用申请磁盘后会自动识别路径，应用不需要做改动

* HPA只扩容不缩容，只扩不缩因为beacon自定义指标qpm取不到值，进入了保护模式，不允许缩容了

* 拉取代码提示blocked，gitlab再登录一下就行了

* 合并的时候可以直接勾选delete source branch 删分支，代码比较的时候是看commit，合并的时候把commit squashed 了，所以source branch 的commit不在target branch里，此时会被视为未MR

* 时区会影响captain页面显示问题

* captain 堡垒找不到实例是soa实例没注册上

* reset 接口目前不对外 只给NOC

* gitlab的compare version功能 在同一个分支下只可比较增量，即source晚于target，否则会显示空白，显示空白时交换source与target即可

* deploy配置自动合并，Settings-->Integrations-->Maven Deploy-->configure-->Ff check and merge

* artifact 默认过期时间为1周，可以自行在pipeline里修改过期时间

* process.env 是 nodejs 取环境变量的方法，需要修改变量时可以直接在CI/CD Variables里修改

* CI/CD 里面可以看到可用的runner tag列表，需要修改runner时可以参考

* node发布无日志，报错path:/api/build.json，需要降级node版本

* SLB的path是指只能访问服务以path开头的路由，如设置it,则只能访问it开头的路由 无法访问vi，可以建多个入口，以域名和path阻断来控制服务功能的提供

* SLB的member port是点火端口【服务端口】，需要保持一致

* 集群只有一台机器，虽然会先拉起一台机器发布，但堡垒机发布完成后，老机器也就开始销毁了，就导致集群内没有提供服务的机器了，从而出现502。

* SLB下线入口，可以看SLB日志，判断TrafficGroups问题

* job重试500，pipeline重试500，是流水线改过

* captain配置的自动mr用户，默认为审批生产的用户，不是发布的用户

* dump文件不能用zip解压，需要用tar解压，否则分析会报错

* hpa自动纳管会修改最小实例数量

* 开启atrust会导致DNS解析错误的问题，如连接wifi等操作

* Cache Node Modules是install 的 缓存，如同一个commit 的话， 缓存的key一致， install 的jog就不执行了，cache的 id 不一致就会执行

* gitlab的Administrator用户可以手动邀请到工程中，需要超管邀请，普通owner invite功能禁用了

* 版本冲突，不要用排除，用dependencyManagement，不是parent依赖的优先级高，而是parent的是dependencyManagement，子项目的是直接依赖的间接依赖

* paas缩容 CDOSError: Inconsistent pods count, in paas: 1, in cdos: 3，点击Force Fetch

* Captain没有批量移交owner的功能，需要一个个申请移交

* HPA扩容是不看单个实例CPU的，看的是集群实例CPU的平均值，流量分配如果正常，请求会均分到集群内的其他机器。毕竟如果其他实例没有cpu也还要扩容，扩出来的实例一样吃不到流量，没有意义，首先现有机器的cpu就没吃满的话，说明现有的流量分流就还有优化空间

* mirror镜像需要reset原group的实例才会有流量

* maintainer权限不能merge或重试流水线，源分支是保护分支，需要添加下人员

* git token 401可以看下是否没有上传公钥

* hotelInfer未绑定appid会导致不生成报告且不执行job

* gitlab加审批权限，直接申请owner即可

* gitlab的mr可以一直提交commit，会自动更新进去，approve只有在创建MR的时候会提示

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

* 发布规范中非dr group不需要等待

* 发布系统单实例情况下会先启一台再销毁，多实例是同时启动和销毁的，批次内可能会导致不可用，若两个实例且不同zone时，会同时发布，会导致服务不可用，可以使用金丝雀来保证可用（金丝雀只发一台）

* 六七年没改的应用竟然报错NoClassDefFoundError，原因是 信安扫描会导致NoClassDefFoundError

* gitlab的机器是独占的8C16G，不会有资源竞争，超了限制会 oom，其他 pod 也一样

* project 权限不能大于group权限，project不能设置public，需要先修改group权限

* PATH VALIDATION-Path priority range cannot be reduced. Modify priority of entryy (3785363,flights/as$) to 1101 or entry (21763,flights/) to 900."}，path和其他group有冲突，调高优先级

* 滚动发布卡住注意等待时长，新建变更时可以配置

* 滚动发布批次内是实时更新的，若发生HPA扩容，会把扩容的也更新成新的镜像

* Captain有慢启动，但SOA是关闭预热的，应用有已下线的SOA服务曾经开启过预热

* npm build超时Listen IPC Channel Failed,kill child process，一般重试就好了。如果想避免的话可以试试锁定esbuild 版本，npm list esbuild查看 nextjs的依赖具体咨询nodejs热线

* gitlab密码错误被锁定只能用户自己登陆解锁

* VM从应用A迁移到应用B找xh或者ysy，需先确认有无入口

* SLB 502是应用问题 需要自查，mirror集群默认拉出 不可用SLB

* Direct member是项目owner，拥有审批权限

* golang UT没有foundtion目录，加下环境变量PAAS_APP_APPID=${APPID}

    ```perl
    export GOLANG_PROTOBUF_REGISTRATION_CONFLICT=ignore && export PAAS_APP_APPID=310001091 && export SOA_SERVICE_ID=30318.htluserquestionservicefunction && go test -v -cover -coverprofile=cover.profile -coverpkg=$(go list ./... | xargs | sed 's/ /,/g') $(go list ./...) 2>&1 | go-junit-report -set-exit-code > junit.xml || true
    ```

* 跑的MR之后的master分支，captain审批的报告却是源分支的报告，因为红线配置的MR，再配置BRANCH即可显示master分支的报告，且使用master分支的执行结果

* ScaleDown, deleting pod for node scale down 是宿主机缩容

* changes展示不完全，点 plain diff，不行那就本地看下吧，changes 太多，gitlab 会渲染不出来，无解

* CDOSError: Inconsistent pods count, in paas: 0, in cdos: 1 找paas二线

* revert会导致rebase失败，需要使用reset回滚

* 如何分辨堡垒流量和生产流量，在bat的trace中若搜到canary.gateway.fx.com或者CanaryRequest=true就是堡垒流量，关键字canary

* 非核心应用是并且设计不需要dr ,是不会告警的

* fat_version_operator,UAT环境审批角色申请组织维度，https://git.dev.sh.com/captain/captain-cd/-/issues/2295

* npm gitlab已部署 但是仓库找不到包，npm 地址切换了，现在gitlab 上跳转地址还未跟进修改【http://npm.release.com改成了https://artifactory.release.com】，实际上是已经部署成功了的，按这个文档配置https://node.fx.com/docs/advanced/develop/artifactory/

* npm ERR! need auth You need to authorize this machine using `npm adduser，node版本设置不正确需要18或20 或者仓库里有.npmrc文件（要删掉）http://conf.com/pages/viewpage.action?pageId=2569510507

* Gitlab MR页面通过idev-xxx分支命名方式可以关联idev了

* SLB PATH冲突的报错显示的是slb groupid，RB可以建XY的入口，XY找不到时可以取RB找

* 发布SHA-ALI不需要同步镜像到公有云

* cpu这类动态指标，每30秒都会计算出所需的副本数，如果算出来的数比你实际副本数高，就会扩容。同时，每个计算出的记录会保持一段时间，时长就是缩容等待时间

* 公有云group，captain展示是一个zone 但实际底层可能是多个zone ，可以看hp 【APM-SYS】的【smarthpa_controller_metric_status_desired_replicas_value】指标的【k8s_clustert】tag

* 不想要某类告警可以 告警屏蔽，参考http://conf.com/pages/viewpage.action?pageId=1082268120

* gitlab点击页面500，可以首页登录一下https://git.dev.sh.com/

* gitlab的deploy没有api接口，不提供

* 商业版，我们gitlab使用的是开源的社区版，reviewer都是ee版功能，approver是我们二开支持的，reviewer暂不支持多人

* 自动merge 是将你发布的代码合到 release 上，不允许 push 会merge失败

* 获取应用信息，osg/ 搜索 CMSGetApp

* Source branch is protected from force push，保护分支不允许 force push

* cdos的问题找容器组

* 后面用了这个IP的实例，有gc，不是自己应用的实例

* HPA，一个az内放不下会放在另一个az，一个区如果挂了，会优先保证实例数，会放在另一个区里

* nodejs build报错 ERR_PNPM_TARBALL_INTEGRITY   http://conf.com/display/RDVacation/2024-01-02

* nodejs build报错kill child process 一般重试就好了。如果想避免的话可以试试锁定esbuild 版本

* arthas需要应用管理员权限和机器登录权限，否则会报错disconnected

* cmsNotReady可以直接reset

* 员工在使用Git推送代码时遇到403错误，尝试通过命令清除凭据但未成功。经过排查，发现问题是由于更改了域账号密码，需要在Mac电脑的钥匙串访问中删除相关密钥。

* gitlab 已经有个名为 feature 分支了，不能再以 feature 开头了

* 开通流水线404，user not found是username与域账号不一致，需要修改username

* cicode and ip do not match 找naqinglu 提供groupid

* 这个MR把master分支的其他commit覆盖了，为什么合并的时候没有冲突提示，这种问题一般是其他用户把master 合并到他们自己的分支，解决冲突的时候丢弃了代码

* 自定义镜像ENV设置的环境变量通过宿主机 exec 到容器里能看到，通过 jump 登陆进去是看不到的，实际上相应环境变量都是有的，只是jump 登陆 不可见，下面的命令需要使用deploy用户才可以执行

    ```perl
    cat -e /proc/`ps aux | grep java | grep -v grep | awk '{print $2}'`/environ | sed 's/\^@/\n/g' 
    ```

* L4LBNotReady，l4lb的member port 在机器上没有启动成功

* 发布时check_fastword报错404，关闭服务集市里的发布前分支检查就行了

* SLB的健康检测地址要写上path前缀

* 跳过重复类检测

    ```xml
    <properties>
    <enforcer.qBanDuplicateClasses.skip>true</enforcer.qBanDuplicateClasses.skip>
    </properties>
    ```

* atthas报错 disconnected: websocket: bad handshake，需要应用管理员权限，若有权限还是不行，需要用命令行直接启动arthas 

* gitlab 关闭AI settings > general > AI > AI in CI report 勾掉就可以关闭了,【手动只能触发一次,自动是增量code review 】

* 本地npm login 401 使用npm login --auth-type=web

* gitlab的cache从生成开始 30天强制过期，用户侧无法控制

* maven goal的覆盖方法不是重新定义一个plugin 会叠加，需要使用如下配置覆盖phase=none

    ```xml
    <phase>none</phase>
    ```

* 使用离线模式会报错【Please supply original non-instrumented classes】需要把prepare-agent 去掉 

* Captain的Api http://conf.com/pages/viewpage.action?spaceKey=CIS&title=CMSGetApp

* eslint-report.json只能上传一个，多个会被覆盖，issue是解析这个文件

* 自定义job的镜像构建失败 需要注意WAR_DIR的变量

* npm run build && npm prune --production执行这个卡住，pnpm 安装的，执行npm prune 可能会有问题，在npm run build前面加一个空格保存就行了

* Spring Boot项目升级JDK21，Jmockit升级1.52.0-jdk21版本之后，UT注解@Injectable注入失败的问题？ 需要把@Injectable改成@Tested(fullyInitialized = true) 

* 设置except ref的情况，把所有job 的 only refs 中的 merge_requests_or_push 改成 branches，MR的触发条件会无视except，优先级较高

* captain 入口删除 申请已存在的情况下可以回滚后重新删除

* 申请权限没收到审批的情况下【如qunar的人员】，需要发邮件申请，项目 owner hui.zhanga@trip.com、anyangliu@trip.com，抄送下 rdscm@trip.com，说明下 申请项目允许开放给 qunar，并说明下申请权限，让 owner 回复下同意，二线这边加下 

* sonar出现版本不兼容问题，http://docs.cloud.com/docs/CICD/code-quality/#sonar-job-%E5%87%BA%E7%8E%B0-could-not-find-artifact-comsuntoolsjar18-at-specified-path-usrjavajdk11libtoolsjar

* maven 3.8.5 有个多线程构建的 bug 【Cannot invoke "org.apache.maven.project.MavenProject.getDependencies()" because "project" is null】，目前正在升级 maven

* Java应用没启动成功且日志看不到报错的情况下，可以查看下是否存在block线程

# FAQ

## 请教一个问题，依赖引入了两个版本不一样的包，但是compile却通过了

maven 默认有依赖冲突解决的 ，如果安装的不是低版本 并且低版本没有配置拦截规则的话，compile job 里的 enforcer 检查插件拦不到低版本，拦截规则，申请拦截规则（maven 默认有依赖冲突解决的 ，如果安装的不是低版本 并且低版本没有配置拦截规则的话，compile job 里的 enforcer 检查插件拦不到低版本，拦截规则）

这个报错应该是 maven 默认解决不了版本冲突的问题，本地 mvn compile 试下，想知道生效的是哪个可以搜 maven 依赖冲突解决策略

# 待办事项

webhook在gitlab的使用

自定义job+自定义镜像

策略应对脚本：机器人，conf，历史会话并发，其中conf可手动

```bash
* Tags： __name__=smarthpa_controller_metric_status_desired_replicas_value appid=100054096 az=SHA-ALI-M bu=SYS container_name=main custom_filter=r100054096-91058148 endpoint_name=beacon endpoint_type=prometheus fqdn=smarthpa-controller-69d656c554-9hkg5.smarthpa.pod.share-sha-ali-pro1.k8s.com groupid=91058148 host=VMSALI801457 idc=SHA-ALI ip=10.25.27.134 k8s_cluster=share-sha-ali-pro1 metric_extra=cpu metric_name=cpu-r100054096-91058148 metric_type=Resource name=r100054096-91058148 namespace=pro-captain object_kind=container orgid=58 paas_type=app region=SHA-ALI workname=r100054096-91058148-79c6cfb575 ~db=APM-SYS

* Tags： __name__=smarthpa_controller_metric_status_desired_replicas_value appid=100054096 az=SHA-ALI-M bu=SYS container_name=main custom_filter=r100054096-91058148 endpoint_name=beacon endpoint_type=prometheus fqdn=smarthpa-controller-864fc46fc9-n2z5h.smarthpa.pod.share-sha-ali-pro2.k8s.com groupid=91058148 host=VMSALI812113 idc=SHA-ALI ip=10.145.210.103 k8s_cluster=share-sha-ali-pro2 metric_extra=cpu metric_name=cpu-r100054096-91058148 metric_type=Resource name=r100054096-91058148 namespace=pro-captain object_kind=container orgid=58 paas_type=app region=SHA-ALI workname=r100054096-91058148-79c6cfb575 ~db=APM-SYS
```

```bash
* Tags： __name__=smarthpa_controller_metric_status_desired_replicas_value appid=100054096 az=SHA-ALI-M bu=SYS container_name=main custom_filter=r100054096-91058148 endpoint_name=beacon endpoint_type=prometheus fqdn=smarthpa-controller-69d656c554-9hkg5.smarthpa.pod.share-sha-ali-pro1.k8s.com groupid=91058148 host=VMSALI801457 idc=SHA-ALI ip=10.25.27.134 k8s_cluster=share-sha-ali-pro1 metric_extra=cpu metric_name=cpu-r100054096-91058148 metric_type=Resource name=r100054096-91058148 namespace=pro-captain object_kind=container orgid=58 paas_type=app region=SHA-ALI workname=r100054096-91058148-79c6cfb575 ~db=APM-SYS
DateTime	TimeStamp	Value
2024-09-05 11:15:09	1725506109.964	1
2024-09-05 11:15:19	1725506119.965	3
2024-09-05 11:15:29	1725506129.964	3
2024-09-05 11:15:39	1725506139.964	3
2024-09-05 11:15:49	1725506149.965	2
2024-09-05 11:15:59	1725506159.964	2
2024-09-05 11:16:09	1725506169.964	2
2024-09-05 11:16:19	1725506179.965	1
2024-09-05 11:16:29	1725506189.964	1
2024-09-05 11:16:39	1725506199.964	1
2024-09-05 11:16:49	1725506209.965	1
2024-09-05 11:16:59	1725506219.964	1
2024-09-05 11:17:09	1725506229.964	1
2024-09-05 11:17:19	1725506239.964	2
2024-09-05 11:17:29	1725506249.964	2
2024-09-05 11:17:39	1725506259.965	2
2024-09-05 11:17:49	1725506269.965	1
2024-09-05 11:17:59	1725506279.964	1
2024-09-05 11:18:09	1725506289.964	1
2024-09-05 11:18:19	1725506299.964	2
2024-09-05 11:18:29	1725506309.964	2
2024-09-05 11:18:39	1725506319.964	2
2024-09-05 11:18:49	1725506329.964	2
2024-09-05 11:18:59	1725506339.964	2
2024-09-05 11:19:09	1725506349.965	2
2024-09-05 11:19:19	1725506359.965	2
2024-09-05 11:19:29	1725506369.965	1
2024-09-05 11:19:39	1725506379.965	1
2024-09-05 11:19:49	1725506389.964	1
2024-09-05 11:19:59	1725506399.964	1
2024-09-05 11:20:09	1725506409.964	1
2024-09-05 11:20:19	1725506419.965	1
2024-09-05 11:20:29	1725506429.964	1
2024-09-05 11:20:39	1725506439.964	1
2024-09-05 11:20:49	1725506449.978	1
2024-09-05 11:20:59	1725506459.964	1
2024-09-05 11:21:09	1725506469.964	1
2024-09-05 11:21:19	1725506479.965	1
2024-09-05 11:21:29	1725506489.964	1
2024-09-05 11:21:39	1725506499.965	1
2024-09-05 11:21:49	1725506509.964	3
2024-09-05 11:21:59	1725506519.965	3
2024-09-05 11:22:09	1725506529.965	3
2024-09-05 11:22:19	1725506539.964	1
2024-09-05 11:22:29	1725506549.964	1
2024-09-05 11:22:39	1725506559.964	1
2024-09-05 11:22:49	1725506569.964	1
2024-09-05 11:22:59	1725506579.965	1
2024-09-05 11:23:09	1725506589.964	1
2024-09-05 11:23:19	1725506599.964	1
2024-09-05 11:23:29	1725506609.964	1
2024-09-05 11:23:39	1725506619.964	1
2024-09-05 11:23:49	1725506629.964	1
2024-09-05 11:23:59	1725506639.965	1
2024-09-05 11:24:09	1725506649.964	1
2024-09-05 11:24:19	1725506659.964	1
2024-09-05 11:24:29	1725506669.964	1
2024-09-05 11:24:39	1725506679.965	1
2024-09-05 11:24:49	1725506689.965	1
2024-09-05 11:24:59	1725506699.965	1

* Tags： __name__=smarthpa_controller_metric_status_desired_replicas_value appid=100054096 az=SHA-ALI-M bu=SYS container_name=main custom_filter=r100054096-91058148 endpoint_name=beacon endpoint_type=prometheus fqdn=smarthpa-controller-864fc46fc9-n2z5h.smarthpa.pod.share-sha-ali-pro2.k8s.com groupid=91058148 host=VMSALI812113 idc=SHA-ALI ip=10.145.210.103 k8s_cluster=share-sha-ali-pro2 metric_extra=cpu metric_name=cpu-r100054096-91058148 metric_type=Resource name=r100054096-91058148 namespace=pro-captain object_kind=container orgid=58 paas_type=app region=SHA-ALI workname=r100054096-91058148-79c6cfb575 ~db=APM-SYS
DateTime	TimeStamp	Value
2024-09-05 11:15:10	1725506110.69	3
2024-09-05 11:15:20	1725506120.691	3
2024-09-05 11:15:30	1725506130.69	3
2024-09-05 11:15:40	1725506140.69	2
2024-09-05 11:15:50	1725506150.69	2
2024-09-05 11:16:00	1725506160.69	2
2024-09-05 11:16:10	1725506170.69	1
2024-09-05 11:16:20	1725506180.69	1
2024-09-05 11:16:30	1725506190.69	1
2024-09-05 11:16:40	1725506200.69	3
2024-09-05 11:16:50	1725506210.69	3
2024-09-05 11:17:00	1725506220.69	3
2024-09-05 11:17:10	1725506230.69	3
2024-09-05 11:17:20	1725506240.69	3
2024-09-05 11:17:30	1725506250.69	3
2024-09-05 11:17:40	1725506260.69	2
2024-09-05 11:17:50	1725506270.69	2
2024-09-05 11:18:00	1725506280.727	2
2024-09-05 11:18:10	1725506290.69	1
2024-09-05 11:18:20	1725506300.69	1
2024-09-05 11:18:30	1725506310.69	1
2024-09-05 11:18:40	1725506320.69	1
2024-09-05 11:18:50	1725506330.69	1
2024-09-05 11:19:00	1725506340.691	3
2024-09-05 11:19:10	1725506350.69	3
2024-09-05 11:19:20	1725506360.694	3
2024-09-05 11:19:30	1725506370.69	1
2024-09-05 11:19:40	1725506380.69	1
2024-09-05 11:19:50	1725506390.69	1
2024-09-05 11:20:00	1725506400.69	1
2024-09-05 11:20:10	1725506410.69	1
2024-09-05 11:20:20	1725506420.69	1
2024-09-05 11:20:30	1725506430.69	1
2024-09-05 11:20:40	1725506440.69	1
2024-09-05 11:20:50	1725506450.69	1
2024-09-05 11:21:00	1725506460.69	1
2024-09-05 11:21:10	1725506470.695	1
2024-09-05 11:21:20	1725506480.69	1
2024-09-05 11:21:30	1725506490.69	2
2024-09-05 11:21:40	1725506500.69	2
2024-09-05 11:21:50	1725506510.69	2
2024-09-05 11:22:00	1725506520.691	1
2024-09-05 11:22:10	1725506530.693	1
2024-09-05 11:22:20	1725506540.69	1
2024-09-05 11:22:30	1725506550.69	1
2024-09-05 11:22:40	1725506560.69	1
2024-09-05 11:22:50	1725506570.69	1
2024-09-05 11:23:00	1725506580.69	1
2024-09-05 11:23:10	1725506590.69	1
2024-09-05 11:23:30	1725506610.69	1
2024-09-05 11:23:40	1725506620.69	3
2024-09-05 11:23:50	1725506630.69	3
2024-09-05 11:24:00	1725506640.69	3
2024-09-05 11:24:10	1725506650.69	3
2024-09-05 11:24:20	1725506660.693	3
2024-09-05 11:24:30	1725506670.69	3
2024-09-05 11:24:40	1725506680.71	2
2024-09-05 11:24:50	1725506690.696	2
2024-09-05 11:25:00	1725506700.69	2

```

# k8s

### 节点

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

### 管理内存、CPU

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

### 调试

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

set -x

for f in /opt/container/entrypoint.d/*; do
    case "$f" in
        *.sh)   echo "$0: running $f"; . "$f" ;;
        *.py)   echo "$0: running $f"; python "$f" ;;
        *)      echo "$0: ignoring $f" ;;
    esac
    echo
done

exec /usr/bin/supervisord -c /etc/supervisord.conf
```

## server.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<!-- Note:  A "Server" is not itself a "Container", so you may not
     define subcomponents such as "Valves" at this level.
     Documentation at /docs/config/server.html
 -->
<Server port="${port.shutdown.server}" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
    <!-- Security listener. Documentation at /docs/config/listeners.html
    <Listener className="org.apache.catalina.security.SecurityListener" />
    -->
    <!--APR library loader. Documentation at /docs/apr.html -->
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
    <!-- Prevent memory leaks due to use of particular java/javax APIs-->
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

    <!-- Global JNDI resources
         Documentation at /docs/jndi-resources-howto.html
    -->
    <GlobalNamingResources>
        <!-- Editable user database that can also be used by
             UserDatabaseRealm to authenticate users
        -->
        <Resource name="UserDatabase" auth="Container"
                  type="org.apache.catalina.UserDatabase"
                  description="User database that can be updated and saved"
                  factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                  pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>

    <!-- A "Service" is a collection of one or more "Connectors" that share
         a single "Container" Note:  A "Service" is not itself a "Container",
         so you may not define subcomponents such as "Valves" at this level.
         Documentation at /docs/config/service.html
     -->
    <Service name="Catalina">

        <!--The connectors can use a shared executor, you can define one or more named thread pools-->
        <!--
        <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
            maxThreads="150" minSpareThreads="4"/>
        -->


        <!-- A "Connector" represents an endpoint by which requests are received
             and responses are returned. Documentation at :
             Java HTTP Connector: /docs/config/http.html
             Java AJP  Connector: /docs/config/ajp.html
             APR (HTTP/AJP) Connector: /docs/apr.html
             Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
        -->
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
        <!-- A "Connector" using the shared thread pool-->
        <!--
        <Connector executor="tomcatThreadPool"
                   port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
        -->
        <!-- Define a SSL/TLS HTTP/1.1 Connector on port 8443
             This connector uses the NIO implementation. The default
             SSLImplementation will depend on the presence of the APR/native
             library and the useOpenSSL attribute of the
             AprLifecycleListener.
             Either JSSE or OpenSSL style configuration may be used regardless of
             the SSLImplementation selected. JSSE style configuration is used below.
        -->
        <!--
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   maxThreads="150" SSLEnabled="true">
            <SSLHostConfig>
                <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                             type="RSA" />
            </SSLHostConfig>
        </Connector>
        -->
        <!-- Define a SSL/TLS HTTP/1.1 Connector on port 8443 with HTTP/2
             This connector uses the APR/native implementation which always uses
             OpenSSL for TLS.
             Either JSSE or OpenSSL style configuration may be used. OpenSSL style
             configuration is used below.
        -->
        <!--
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
                   maxThreads="150" SSLEnabled="true" >
            <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
            <SSLHostConfig>
                <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                             certificateFile="conf/localhost-rsa-cert.pem"
                             certificateChainFile="conf/localhost-rsa-chain.pem"
                             type="RSA" />
            </SSLHostConfig>
        </Connector>
        -->

        <!-- Define an AJP 1.3 Connector on port 8009 -->
        <!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->


        <!-- An Engine represents the entry point (within Catalina) that processes
             every request.  The Engine implementation for Tomcat stand alone
             analyzes the HTTP headers included with the request, and passes them
             on to the appropriate Host (virtual host).
             Documentation at /docs/config/engine.html -->

        <!-- You should set jvmRoute to support load-balancing via AJP ie :
        <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
        -->
        <Engine name="Catalina" defaultHost="localhost">

            <!--For clustering, please take a look at documentation at:
                /docs/cluster-howto.html  (simple how to)
                /docs/config/cluster.html (reference documentation) -->
            <!--
            <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
            -->

            <!-- Use the LockOutRealm to prevent attempts to guess user passwords
                 via a brute-force attack -->
            <Realm className="org.apache.catalina.realm.LockOutRealm">
                <!-- This Realm uses the UserDatabase configured in the global JNDI
                     resources under the key "UserDatabase".  Any edits
                     that are performed against this UserDatabase are immediately
                     available for use by the Realm.  -->
                <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                       resourceName="UserDatabase"/>
            </Realm>

            <Host name="localhost"  appBase="empty"
                  unpackWARs="true" autoDeploy="true">
                <Context path="${vdir.server}" docBase="${docbase.server}" reloadable="false"/>
                <!-- SingleSignOn valve, share authentication between web applications
                     Documentation at: /docs/config/valve.html -->
                <!--
                <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
                -->

                <!-- Access log processes all example.
                     Documentation at: /docs/config/valve.html
                     Note: The pattern used is equivalent to using pattern="common" -->
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

# 公有云

## AWS

### [AWS NLB Failopen](http://conf.com/display/CIS/AWS+NLB+Failopen)

踩坑点：AWS的NLB以LB级别作为FailOpen机制的生效对象，如果NLB监听端口后端的任一TargetGroup在已启用的可用区中没有运行状况良好的目标，AWS会从 DNS 中删除相应子网的 IP 地址，以便请求无法路由到该可用区中的目标。如果在所有已启用的可用区中，所有目标同时未通过运行状况检查，则负载均衡器将在失败时开放。失败时开放的效果是允许传输到所有已启用的可用区中的所有目标的流量，而不考虑这些目标的运行状况。

相关官方文档：https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html



故障记录：因为配置NLB时为了能够快速满足可能产生的单AZ资源不足的情况下扩容至其他AZ，所以3个AZ的subnet都enable了，在稳态下虫洞只有2个AZ内有部署EC2，利用NLB 的检测逻辑【If target groups don't have a healthy target in an enabled Availability Zone, we remove the IP address for the corresponding subnet from DNS so that requests cannot be routed to targets in that Availability Zone】确保只会解析出我们部署了服务的AZ IP能解析出来，也就是虫洞的稳态运行模式。

在1/11 晚上虫洞NLB新增9080端口后 因为sg拦截 导致health check失败了然后落到了我们预期之外的fail open逻辑里【If all targets fail health checks at the same time in all enabled Availability Zones, the load balancer fails open. The effect of the fail open is to allow traffic to all targets in all enabled Availability Zones, regardless of their health status.】此次操作预期会和IDC同样的逻辑一样，新增listener哪怕无法通讯也不会影响到原有的listener故未通知NOC，操作后因为NLB的fail open模式不会再内部详细检测NLB内是否有其他target group正在正常运行，直接以NLB级别进行了fail open，所以导致出现了未部署服务的AZ IP出现在解析IP里。在操作时没有预期到此操作云上云下逻辑不同导致了之后的业务虫洞故障。



### AWS Target Group PortAllocationCount 限制

**背景：**

IBU同学反馈QUIC AWS-SIN访问H5 Gateway有大量5XX报错，同时查看APM监控发现服务成功率从99%抖动下降至约94~92%

**链路拓扑：**

**排查过程：**

经NOC确认异常时间点前后无变更，查看QUIC Server 和SLB Private-service NLB指标，QUIC NLB监控指标观测无异常和历史曲线无差异。

SLB Private-service指标New flow count和 Client reset count有明显上升

由于订单未反馈异常也没有变更需要回滚，直接在线排查：

1、同Region其他NLB实例指标，未见异常，监控大盘也未观测到某个AZ大批量异常指标出现，**排除AZ级别问题或网络抖动**；

2、NLB**已有**收集监控指标，未见明显和过往曲线对比的异常抖动；

3、Target Group EC2监控指标 ：Hickwall 查看QUIC Server和SLB Server无异常，整体使用率和过往曲线对比没有明显抖动，**确认磁盘IO也未达到上限**；

4、框架根据历史过往希望确认是否出现NLB fail open问题，查看target group health hosts count值确认无上下浮动，**排除fail open问题**；

一小时后指标自动回归正常指标线，但在2天后复现，同时发现虫洞请求SLB Private-service也有5xx数值上升，开CASE给到AWS，并提交了相关链路上的设备id信息，AWS查询后发现

PortAllocationErrorCount 指标异常和成功率下降的时间点很耦合，定位在SLB Private-service的NLB上。

https://docs.aws.amazon.com/zh_cn/elasticloadbalancing/latest/network/load-balancer-cloudwatch-metrics.html#load-balancer-metrics-nlb

复核AWS的文档发现，在Target Group内的每台EC2，在使用地址转换获取用户源IP时，有每台机器支持与每个唯一目标（IP 地址和端口）的 55,000 个并发连接或每分钟约 55,000 个连接的限制。

这套NLB因为target group内的EC2有请求本身CNAME的需求，所以使用了proxy protocol来获取用户源IP，在TCP包内看到的源IP均为NLB的IP地址。

该Target Group内部署EC2为4台 55000*4=220000，NLB的ActiveFlowCount约30000，NewFlowCount大约180000，正好在峰值时达到了阈值上限。

确认问题点后框架团队先将SLB的Target Group扩容至8台，并和IBU QUIC团队沟通新增请求数数值过大原因。

经QUIC团队确认会有一个自建Nginx，通过upstream方式将请求转发至SLB，但是在转发时原先配置不会使用长连接保持，而是每个请求直接开启一个新连接，所以导致NewFlowCount数值较大，

QUIC团队之后修改配置并发布后NewFlowCount降至原先的5%

### NLB enable子网和target group接流逻辑(2023/12/28 RCA)

背景 -- 预期将FRA-AWS所有NLB均开齐 3AZ subnet，让负载均衡流量更均衡并避免单个AZ EC2资源不足的情况下可以使用其他AZ资源的需求

时间线：

2020/2/19  15:32 CMS Group创建成功，创建时自动接流状态值是true

2020/2/19 15:48 NLB创建

2023/9/18 Group扩容，由于该CMS group在当时发布自动接流为true直接注册了进了NLB的target group(实际未部署完成)，webinfo状态显示为健康检测失败，calabash数据库中状态为注册成功，AWS Target group因为AZ未enable显示为unused。

2023/9/18+ SLB团队发现发布即接流后，修改CMS 修改接流字段值从true改为false

2023/12/28 16:48 NLB 开启 AZ-B子网 GroupID:21040345 arn:aws:elasticloadbalancing:eu-central-1:671660153913:loadbalancer/net/fraaws-private-webapi-s-slb/26f4cd91a214d5d4

2023/12/28 16:48 开启3AZ的后target group中的ec2数量从正确的6台机器 引入了3台AZ B的未投产EC2 改为了9,然后短暂跌0，再次回升到9

2023/12/28 16:53  应用报错量上升

2023/12/28 18:00 火车票反馈请求虫洞异常

2023/12/28 18:19 拉入电话会议

2023/12/28 18:26 反注册AZ-B 3台未投产机器的恢复故障

2023/12/28 18:27 故障恢复



疑问：

1、为什么其他0716中的group没有一样的问题？A:抽查了其他几个CMS Group 自动发布接流的值均为false，目前已同步框架团队复核cms group信息是否均正确为自动发布接流值 false

2、其他两个AZ的EC2为什么没有同样的问题？

3、如果三台未投产机器一直在target group内但处于未enable子网的ec2 在开启crossAZ流量后是否会接流？ A:经过实验验证 crossaz后不会接流，需要子网enable才会走健康检测逻辑确认是否接流

4、如果target group内的ec2 在未enable的子网中 是否会显示count于不正常主机数量？ A：经过实验验证 不会显示在不健康主机count中，只会显示状态为unused

5、三台异常机器的注册和反注册时间 所有变更历史(10.14.6.73,10.14.12.245,10.30.133.248)？ A：09/18注册进入target group 12/28 18:26反注册

6、开启3AZ是否是否会触发target group的member更新？ A：会检查新enabled的子网中是否有unused状态的EC2，并进行健康检测确认是否健康可以接流

7、为什么未部署的服务器会通过健康检测？A：健康检测为4层TCP握手检测，未部署业务EC2虽然未部署完成SLB配置，但是已监听80端口

8、为什么识别告警耗时会需要1小时？A：报错上升整体占比较小，请求链接均为长链接，建联成功可复用100次请求，如果单次请求建联失败会直接重试有66%概率进入正常运行EC2并建立长连接，变更执行行为目标为LB，观测点在LB的监控指标上，未观察Target group监控指标



根因：

9/18 Group扩容后由于CMS默认接流配置，注册进入AWS Target Group中，但是由于子网未enable所以未接流，并且在webinfo中显示拉入失败，框架同事对于这3台EC2在12/28前未进行SLB的服务部署。

9/18+ SLB 扩容拉入后即发现发布就会接流的问题，修改了CMS 自动接流字段值为false，从该时刻起新扩容的机器不会再自动接流，但已注册在target group中的3台 AZ-b的EC2仍保持unused状态。

12/28 的变更启用了NLB新的AZ，target group中AZ-b的EC2由于安装了空nginx并监听80端口，通过了健康检测并接流导致了9台接流的EC2中有三台其实未部署SLB直接返回了异常值给客户端。

## 阿里云踩坑记录

### Endpoint挂载安全组导致偶发请求不可达

参考SGP Endpoint & Endpoint Service 测试方案，测试过程中偶发connection timeout

经过阿里云内部检查，了解到是安全组设置的问题，需要把ep outbound安全组（sg-t4n9qpyb1g9o5zmqp8eb）设置为全开

\---

原话：ep安全组出向配置了 0.0.0.0 /80 443的访问控制，ep在回包的时候命中不了这规则，目标是客户端的IP和TCP高位端口，底层ep的转发设备是多台的， 偶发能通原因是，回包hash到了有session的ep设备上，不经过安全组校验，直接转发
\---
总结：

ep的安全组出入向都作用在发起请求的client

ep有多个'物理'实例，回包如果走不同的ep底层转发设备就需要校验出向规则，导致回包被拒绝（偶发失败的原因）；如果走同一ep底层转发设备，由于同一session，不校验出向规则（偶发成功的原因）

### SGP阿里云海外入口请求延迟和成功率优化

**阿里云东南亚公网质量调优——亚太大区调度池化**

调度池化：

- 跨Region覆盖：通过全部或by用户的调度，让用户就近接入，以高质量专线提升跨Region访问质量。
- Region内覆盖：增加东南亚二级运营商提供质量更可靠的广覆盖能力。

技术原理：基本调度方法：长传+远端公网

- 常态：默认不做调度，出向和入向流量均经公共运营商网络，经 BGP 最优选路，质量结果不确定性稍高
- 出向调度：可以根据目的进行调优，结果确定性强，带宽方面，只需要沿途的长传，公网的出向带宽满足安全水位要求即可
- 入向调度：路由 Anycast 到目标 ISP，效果存在不确定性，除了长传+公网的入向带宽水位之外，需要以路由宣告的方式从ISP 引流，需要EIP 段聚合为段，需要符合运营商接收路由的 prefix 要求



**quic前置阿里云NLB替换为支持udp分片特性的nlb**

tcp链路优化的效果比较明显的同时quic的成功率提升优化效果并没有同样显著，所以和阿里云配合分别在quic server、nlb等网络路径上抓包，进行排查。

在抓包后发现quic server的ECS抓包中发现quic server回包给nlb有分片包，而阿里云nlb默认不支持udp分片，如果需要支持udp分包需要以账号维度进行白名单开白操作。



在开白前的nlb则不会自动支持udp分包，如果对特定instance进行单独配置的话变更下发需要30分钟，变配下发过程中，存量长连接可能跌量，影响大约1min内，新建不受影响。

为避免对生产流量产生影响，并能够明显确认特性差异导致成功率差异，选择新建一个nlb并进行流量灰度验证，最终确认成功率在支持udp分包的nlb上成功率上升至99.5%+

**优化时间及优化内容记录**



**之前NLB未开启UDP分片能力导致成功率相比开启后相差约0.7%的原因大致分析如下：**

由于l4lb-operator创建nlb的规范定义默认关闭保留源IP选项并使用TOA来记录源IP，所以udp包从nlb后端的服务器视角来看所有外部请求进来都是通过同样的3个NLB IP过来的，我们在quic服务器上通过ping和tracepath来检查和nlb之间的mtu值发现值有抖动(长ping包设置1472size之后发现出现时通时不通并且pmtu值一直有变化)



pmtu是用于确定从源主机到目的主机之间路径上最小mtu值的过程，以避免IP分片的工作机制，所有外部真实公网会转化为NLB的IP 所以quic server和nlb之间的pmtu一直在跟随外网的路由包的变化而变化



在未切换到支持udp分片的nlb的情况下：

\#1 app端client的行为，默认没有set df位，如果过nlb的包是分片包return到client会因为组装失败而timeout failed，外网请求nlb的入向包已分片或>1500(pmtu值) 只传首包后续包无五元组信息丢弃

\#2 外网请求nlb的入向包<1500(pmtu值)，正常入向完成业务请求，封装完return包后回到quic server，由于vpc内所有服务器和网络设备mtu值均为1500，回包中data数据大小可能是1472+28包头size

  \#2-1 pmtu值被后续入向网络包刷新为<1500(或小于return包)的值如1360，由于ECS支持分片逻辑会将包分成1500-1360 和后续包，然后NLB返回请求源但是由于NLB不支持分片仅传首包，后续包无法拼接然后组装失败，整体请求失败。

  \#2-2 pmtu值被后续入向网络包未被刷新大于回包size，ecs认为无需分片则正常回包返回请求源，整个请求视为成功

  \#2-3 pmtu值被后续入向网络包刷新，大于请求入向的pmtu值，正常从nlb返回，但是实际路径上设备的mtu值小于包大小，则根据网络设备是否支持分片决定是否能够正常范围(传统网络设备大部分支持udp分片)

\#3 AWS默认nlb支持udp分包行为，其他阿里云客户理论会遇到同样问题，阿里云已做了内部典型case汇报

### 三中心架构下应用跨可用区调度性能下降排查

事件背景：

用户报障服务上云后应用响应速率对比IDC有所下降，从阿里云和RB、XY 的95线及平均值对比来看均有一定幅度的响应速率下降

查询方向：

RB、XY、ALI之间的跨AZ流量延迟差异不大，SOA仅首次访问会在SOA注册中心端，获取服务方IP，之后直接IP调用，排除因NLB请求跨穿越AZ导致的延迟，

下一步查询用户应用的上下游调用链及依赖项，发现应用依赖两个核心Redis均实现了多机房部署，但有一个仅在RB、XY有Redis server在阿里云上没有节点，



查看到该服务在不同IDC之间Redis调用时间差异较大，找到应用让其联系DBA扩容SHA-ALI的节点进行优化尝试

### 应用绑定公网EIP 达到出向限速导致丢包问题记录

应用部署于公有云环境出向公网一般为通过proxy gateway或直接绑定公网EIP出向两条路径，

从paas上申请出来的VM如果申请了公网EIP默认入向速率上限为100M，出向速率为5M。

如果在业务高峰期观察到丢包可以参考以下故障记录：

故障现象：CC团队两台SHA-ALI服务器绑定了EIP，在业务高峰期发现请求有丢包现象。



排障过程：登录服务器尝试向外网服务器进行7层和4层请求测试，发现丢包率约在2%左右。

配合供应商抓包发现，请求失败较多，高峰丢包率达到，同步查看EIP出向速率发现峰值已达到5M，

确定为EIP出向速率限制导致。



在阿里云console上对指定服务器的EIP就行限速变更提高到15M并进行后续观察发现故障消失。

### 阿里云 四层负载均衡 批量拉入拉出失败记录

目前阿里云上使用了CLB并规划了未来使用NLB作为四层负载均衡入口，

公有云负载均衡数据信息同步回IDC的机制为，

1、首次人工介入部署并绑定tag【group】：【$CMSgroupid】

2、calabash每隔10分钟会定时任务同步最新云上负载状态信息至CMS

3、如果在webinfo进行手动拉入拉出操作，会直接触发对应公有云的API进行相关【注册】\【反注册】操作



故障记录：

calabash操作CLB后端服务器的拉入拉出发布时遇到报错如下：

VServerGroupProcessing A previous configuration of the VServer group is pending; please try again later. RequestID: BCE80530-FD13-5C8A-84ED-D90B363XXXXXX

开工单咨询阿里云后确认故障原因是上一个api添加后端服务器的配置未生效，导致移除的api执行失败，对服务器组确实同一秒钟还有AddVServerGroupBackendServers动作，对应的request id9A61BC98-18A4-559C-9622-01225DXXXXXX ，建议单次调用最添加多个ecs，或者间隔一段时间再调用。

由于上游tars会逐个服务器发起request请求，导致并发调用阿里云接口，并产生如上报错。

临时解决方案是：

RouteCenter中对于group属于阿里云IDC的group操作上锁级别从instance调整为group级别，确保group同时发布或拉入拉出时，执行会已group维度上锁。

# 超时治理

**背景：**

携程近几年，出现比较多起由于超时配置不合理的RCA事件。统计在2019-2021年中演练发现超时配置问题在27起，生产由超时引发的3级以上（含3级）事件46件。

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
| 举例：10001000                       | SOA          | international.getlocaleconfig     | 2.21.2                                        | 200ms                          | 1200ms             | 100s                                                         | 5s                        | 5s         | 跳monkey专项演练 |

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