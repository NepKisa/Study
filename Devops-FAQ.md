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

![image-20240227154730416](D:/Code/Notes/devops/captain/Captain.assets/image-20240227154730416.png)

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

* Group名字无法修改

* arex无限重启，关闭了coverage功能后发布成功

* 互斥变更也有APi提示【http://conf.ctripcorp.com/pages/viewpage.action?pageId=1307893240】

* 发布更新时，先拉出再拉入导致流量接不住，可以控制批次缓解，发布是新旧并行的，实例一段时间不可用属于正常现象发布规范不生效

* paas物理机下载失败可以webinfo上重启，重启无用找汪怡

* IP实例消失，但在SOA平台上存在，SOA平台数据异常

* 两个appid创建同一个域名的入口需要path不一样

* 域名无法搜索到appid可能是虫洞域名，详细可咨询SLB热线

* HPA有多个指标，有一个满足就会扩容，缩容时需要全部满足才会缩容，若QPM无数据则无法判断

* arex无限重启（java.lang.NoClassDefFoundError: io/arex/agent/inst/runtime/coverage/CoverageSupport），找arex支持关闭了coverage功能后发布成功

* 流水线权限显示没有owner或maintainer时（git上已配置）可以在captain的流水线列表同步成员权限

* normalop密码，直接申请powerop的权限

* paas物理机下线直接缩容

* SHA-ALI基本信息里的tenant可以看到类型是DEFAULT或TRIPAL

* 机器反复重启，查看下内存CPU指标，可能是机器配置低导致的，sys.cpu.throttled_time是CPU资源不足，调高配置解决

* 更新页面的事件中心是全局的，无论更新的是啥，显示的都是全部的日志，更新hpa与pod无关

* 实例磁盘满了不会自动重启，只能手动进机器清理磁盘恢复或者reset

* JVM参数不设置HeapDumpBeforeFullGC和HeapDumpAfterFullGC是因为太频繁会把磁盘撑满，此参数是在FullGC前后dump

* 点火未进行且bat无任何日志时可以使用arthas，thread -b查看阻塞的线程

* tars相关找杨宇杰

* 8.1.0bom的qconfig会线程阻塞导致发布卡住

* 生产环境SLB和CMS信息未同步找王子健

* L4LB找徐文杰

* 域名绑定找周蒙蒙

* `Module build failed: TypeError: Invalid PostCSS Plugin found`这个报错网上有issue

* UT覆盖率点开403：http://conf.ctripcorp.com/pages/viewpage.action?pageId=1400443158

* UT无法通过需紧急通过可以自定义job使用：allow failure：true

* Node的lint太慢，可以只检查必须的模块，非必要的都配置eslintignore，.eslintigore google一下

* Snapshot包检查：http://conf.ctripcorp.com/pages/viewpage.action?pageId=603516275

* UT宝典：https://git.dev.sh.ctripcorp.com/liangliang.dai/issues/-/issues/72

* 重复类问题：[重复类的应对方法 · smartbook (ctripcorp.com)](https://pages.release.ctripcorp.com/GitLabHome/smartbook/maven/duplicate_class.html)

* 堡垒机发布卡住：可以新建变更重新发布尝试

* 临时关闭UT覆盖率检测：将应用降级为非核心，再删除红线即可

* 代理使用架构图：[代理模式及直连代理使用 - mw莫巍 - Confluence (ctripcorp.com)](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1471572539)

* 曾经发布成功的版本再次发送未必成功，外部组件可能有变化导致发布失败，如qconfig、数据库等

* 下载卡住可以webinfo重启一下服务器，然后等十分钟再发布

* npm deploy时缓存问题可以修改 "prepublishOnly": "rm -rf node_modules/ package-lock.json && npm install && npm run build"

* 如果有浏览器中发起访问外网请求，配置代理https://node.fx.ctripcorp.com/docs/advanced/faq#%E5%A6%82%E6%9E%9C%E5%BA%94%E7%94%A8%E8%A6%81%E8%AE%BF%E9%97%AE%E5%A4%96%E9%83%A8%E5%85%AC%E7%BD%91%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%BA%94%E8%AF%A5%E5%A6%82%E4%BD%95%E8%AE%BE%E7%BD%AE%E4%BB%A3%E7%90%86

* [rename docker image tag without pull image to local](http://conf.ctripcorp.com/display/~yfqin/rename+docker+image+tag+without+pull+image+to+local)

* 删除fork的原项目可以先在general取消fork【owner权限】防止关联删除

* 酒店job：hotelinfer

* SCM new Chinese检查可以在isssue里搜索tag【new Chinese】查看，general里可以看min chinese code

* SCM approval没权限会报错【An error occurred while submitting your approval】，approval by 后面的是需要点的人【还没点】

* SCM deploy失败但maven仓库有【偶然现象。可以寻找二线更改gitlab状态】

* Captain 固定IP，镜像tag不变的情况下会优先使用本地的，务必更新tag[][]【http://conf.ctripcorp.com/pages/viewpage.action?pageId=723903265】

* 资源不足可以reset释放资源

* L4LB（徐文杰）一律打SRE 硬件入口 标签

* 发布系统的堡垒机不接流量，ContainerNotReady不接流量【70s缓冲】，拉入堡垒会接流量【金丝雀】

* 切换root用户sudo -i失败可以用sudo -Es

* 关闭HPA无法扩缩容，删除HPA可以

* 镜像订阅后会自动同步到公有云

* 堡垒发布卡住，要所有实例ready才能成功，可以reset

* 换环境发布失败，有redis报错，检查一下代码不同环境redis是否同名

* 新建变更过程卡住【如卡在滚动发布】，需要所有实例ready才可进行下一步

* fra的group需要二线开白名单才可创建

* 流水线执行请求体过长报错413【find . -name *.pyc -delete】，包太大了，用自定义 job，把 package 和 image 合成一个

* JAVA17 ZGC会导致启动时占用内存极大，设置Xmx与Xms相等即可

* this commit is not fast forward to target branch 配置了自动合并，发布的分支 和 master 分支没法快速合并，发布分支要处理一下，可参考http://conf.ctripcorp.com/pages/viewpage.action?pageId=764421513

* 一个应用可以最多创建的fat环境数量没有限制

* No asset id or system user id found，申请登录权限

* sonar问题找dll，能修就修 不能修就忽略[如何处理-issue-误报](http://docs.cloud.ctripcorp.com/docs/CICD/code-quality/#如何处理-issue-误报)

* tomcat7和jdk17的兼容问题导致点火404，服务启动访问不到vi/health

* oar插件相关可以看http://conf.ctripcorp.com/pages/viewpage.action?pageId=941223971&preview=%2F941223971%2F1463186716%2FCaptainCI%E9%A1%B9%E7%9B%AE%E4%BB%A5%E5%8F%8A%E6%9C%AC%E5%9C%B0%E5%8C%96%E6%9E%84%E5%BB%BApipeline%E6%8F%92%E4%BB%B6Oar%E4%BB%8B%E7%BB%8D.pdf

* gitlab pages可以参考以下四个文档

    > http://conf.ctripcorp.com/pages/viewpage.action?pageId=879142305
    > http://conf.ctripcorp.com/pages/viewpage.action?pageId=626676407&preview=%2F626676407%2F639931137%2F%E5%9F%BA%E4%BA%8EGitlab%E7%9A%84%E6%96%87%E6%A1%A3%E7%AE%A1%E7%90%86%E5%92%8CPages%E6%B5%81%E7%A8%8B.pptx
    > http://conf.ctripcorp.com/pages/viewpage.action?pageId=721158650
    > http://conf.ctripcorp.com/pages/viewpage.action?pageId=1006444652

* 发布规范可以修改DR等待时间

* Mirror有两台机器，流量都在一台机器上，开了hpa导致的 我晚点找人固定一下mirror的zone 这个后续会跟容器那边商讨解决方案，之前都是手动搞的 mirror类型特殊 不需要每个区都保留实例

* 机票审计检查【jira】 问一下邱燕武

* sonar 没有办法去掉，jacoco 可以去掉。机票的规则是所有应用要进行code review，如果用户需要去掉红线，发送邮件到部门pmo，抄送scm团队。pmo 同意了，肖小龙那里可以加白。

* pages服务访问页面404 ，要在public下。可以找李海涛清下缓存

* deploy一直转圈，可以换版本重发，或者让李海涛改成失败重发

* 查看sonar报告接口：https://git.dev.sh.ctripcorp.com/helpcenter/helpcenter/-/merge_requests/468/pipeline_reports?type=sonar&detail=incr

* 出现中文红线时可以考虑申请非国际化：[【Captain】如何申请应用的非国际化 - 公共技术服务中心 - Confluence (ctripcorp.com)](http://conf.ctripcorp.com/pages/viewpage.action?pageId=2320798956)

* gitlab token 新版本只能用个人 token，账号密码不行了

* Node UT失败，settings > ci/cd > general pipeline > Use separate caches for protected branches 勾掉，模板有问题【cache的key不一样】，自定义test，生成的 clover 报告里，覆盖率是0，jest 测试不用 nyc

* 袁刚有报表查看 某个组所有应用每次pipeline的耗时吗

* This merge request is not associated with any iDev issues. 重新绑定下idev需求和分支。建议需求发布后不要继续在同一个分支上开发，最好是建fix分支修改。绑定分支发布后，gitlab 这边会判断 idev 需求已发布，所以不会再展示

* NodeJS Serverless没有红线配置

* go 服务编译 下载组件失败404，http://conf.ctripcorp.com/pages/viewpage.action?pageId=2352370861，http://conf.ctripcorp.com/pages/viewpage.action?pageId=2264739341，改 golang variables ，使用新的 artifactory 服务代理 golang 包，不用 goproxy.release  了，就是要配置一遍 公共账户 和 token 到你仓库 settings variables 里，不懂可以找yyj

* gitlab ci Uploading artifacts as "archive" 413，自定义job，合并成一个job

* 账号被锁问题找李海涛

* 流水线开通：[【SCM】开通流水线 - 公共技术服务中心 - Confluence (ctripcorp.com)](http://conf.ctripcorp.com/pages/viewpage.action?pageId=1722670374)

* sonar规则不是SCM制定的。误报只能行忽略，或者项目忽略

* UT new覆盖率接口：https://git.dev.sh.ctripcorp.com/htlproductmatch/htl-ai-business-controller-service/-/merge_requests/28/pipeline_reports?type=jacoco&detail=incr

* new 覆盖率changes diff 过多会有部分统计不到，这种只有两种方法，补全能看到的，或紧急合并， test job 跑完后，可以访问 https://git.dev.sh.ctripcorp.com/vacation-engine/shopping-service/-/merge_requests/856/pipeline_reports.json?detail=incr&type=jacoco查看详情

* git push不了可以删除凭证看看【git remote -v】？

* 依赖下载不了可以找yyj帮忙下载了上传，【手动下载 jar 然后上传到 host 类型的 3rd repo 仓库】

* sonar 自定义规则不支持用户配置到自己的项目上，不支持自定义个规则怎么把规则配置到gitlab的sonar扫描流程里

* mirror问题咨询PerfLabs Support(性能实验室支持群)

* captain inplaceupdate选项关闭每次发布都会更换容器，否则会原地升级，2024.5.17左右会修复

* 酒店job未断言问题，找严红娟修改规则

* arm和amd镜像的sha256不一样，打镜像时用默认的基础镜像才是多架构的

* mac require路径mac大小写不敏感、Linux敏感

* maven deploy 建议直接升到 1.0.14版本，不然之后部署每次都要找二线改状态

* 批量删除分支Api https://git.dev.sh.ctripcorp.com/help/api/branches.md#delete-repository-branch

* 非标准镜像，登录失败ssh: handshake failed: ssh: unable to authenticate，attempted methods[none publickey] [生产 j 堡垒机 Linux 密钥登录失败自助排障](http://conf.ctripcorp.com/pages/viewpage.action?pageId=2361200591#id-生产j堡垒机Linux密钥登录失败自助排障-非标镜像容器（用户自己构建）)

* sonar不支持21的语法且目前没有升级的打算，会支持 java21 的扫描，但是不会支持这些非常新的参数

* paas 发布单已完成。页面还显示，找czz清理一下

* pass页面有未删除的group、但销毁group按钮是灰色，显示captain删除，可以删除disabled，再点击销毁

* gitlab退出组：https://git.dev.sh.ctripcorp.com/groups/【组名】/-/group_members

* 创建应用500，去paas上创建

* iaas可以强删容器

* Captain创建group 【An error occurred while creating resources during cms sync: V2CreateGroup: idc:QATE1 不能匹配枚举lookup中的 IDC_TYPE】，这边创建https://captainsmoking.release.ctripcorp.com/app

* 点火404，应用没起来，SpringBoot3选9就是这样，它只能用10，SpringBoot 1和2选10也是同理

* 自定义tomcat参数：【java 规范库，和 extraenv.sh 写在一起的 自定义 tomcat server.xml 的方式。和 extraenv.sh 放在一个目录下，就是需要从 容器里先找个 server.xml 完整 copy 下，再修改自定义配置的部分】

* cpuset开启后重新发布才会生效，可查看yaml文件确认resource.limit和resource.request

* 查看 catalina.out 日志可以找到 Spring Boot 的版本。如果是 2 开头的，出现类似的 javax 开头的类 ClassNotFound，并且用户用的是 Tomcat 7，那就让他们升级到 Tomcat 8 试试

* Gitlab移交owner，[移交Owner | 系统研发文档 (ctripcorp.com)](http://docs.cloud.ctripcorp.com/docs/Gitlab/权限管理/移交Owner/)

* tomcat版本问题导致发布点火失败，http://conf.ctripcorp.com/pages/viewpage.action?pageId=894961691

* 现在公有云机器是弹性的 可能会回收 影响固定IP的质量 需要固定IP的话建议注册vm应用

* JMX Compilation，JIT http://conf.ctripcorp.com/display/RDVacation/100032515+full+gc

* 如果合并的代码里包括dev_20240426_trip 分支的mr的更改，会自动合并

* Group entrance is not empty，找czz清理数据

* 令牌检验不通过，itoken/ 测试下令牌，尝试重新绑定

* Java 11 里 ZGC 还不支持 linux/arm64，15以后支持

* git 邮箱检测未生效，回退一步重新提交生成 git commit

* project changes 关闭只能后台处理，后期changes 不做维护和新功能支持了，建议之后转成merge request

* HPA 应用是多集群架构，在xy-b 有2个， xy-c有一个 实例，两个集群的hpa是独立的【指标除2计算，xy-b 这边，平均是大于20的，所以缩不到1】。监控页面，左上k8s_cluster 切集群，挨个看xy-a, xy-b, xy-c 是否有实例

* captain新建应用失败可以登陆下paas再建

* http://idb.ops.ctripcorp.com/#/redis/health-tool redis如果遇到连接问题，可以使用这个工具先排查下，确认下是不是跟网管或者redis本身有什么问题

* SGP Zone B能发成功，Zone C不能发成功【有soa服务找不到的报错】，看着上次 sgp 迁移的时候 路由更新需要手动处理下，和代码版本有关

* captain开通流水线400，仓库不存在，由于仓库改名，可以从git开

* 测试环境扩容是XY的，目前底层已经把其他的关闭了 只有新源了

* SLB 的nginx是1.20,node的nginx是1.16

* 合并到master触发，那只设置ref master 就行了，master 分支上代码有变更就会触发

* 恢复流水线可以找ljh

* only one file can be sent as raw，artifact里clover只能上传一个文件

* No hetel_test report没跑api自动化

* captain自动回收只在fat，可以放到fws

* nodejs SLB访问502，健康检测不通过，健康检测地址修改为/slbhealthcheck.html

* 目的是看看这段时间，是宿主机负载高影响到容器，还是容器负载高同时影响到了宿主机。思路也比较简单，看看这段时间宿主机上面的所有容器监控，如果容器监控其他没问题，同时也没有新容器调度到该机器，仅一个 容器有问题，那还是容器影响到了宿主机

* idev QA测试和镜像发布者不能是同一个人

* captain war包链接7天就过期

* paas机器数量最多三台 全局限制

* paas迁移，http://conf.ctripcorp.com/pages/viewpage.action?pageId=394798188

* 同步入口CR卡住，可以点暂停再点继续，或者重新发布

* 审批已通过，流程没往下走，因为等了太久  captain这边先给停掉了，下次这种情况等审批过后自己在变更详情页面再点一下申请审批就好了

* paas显示发布中，但是发布记录找不到，状态没同步回来，用相同版本覆盖发布一次这个group

* it Helpdesk 申请 SCM 公共账号

* MR的时候，同commit结果会覆盖，可以重新collect刷新

* sonar的中文检测不检测注释，修改等同于新增，nosonar可以对中文检测生效

* 自定义镜像，http://conf.ctripcorp.com/pages/viewpage.action?pageId=677104586

* paas发布失败无日志，webinfo重启服务器，十分钟后重新发布

* UT退出137可以使用redis service

* MR没有revert按钮，fast-forward 没有生成merge commit,没有办法直接revert 整个mr，只能一条一条revert

* maven deploy是使用的上一次成功的配置，更改settings后多试几次

* Java类型的容器可以通过指定基础镜像来实现tomcat

* 自定义pipeline需要这个变量 DOCKER_TLS_CERTDIR: ""，以下配置缺一不可

    ```yaml
    .dind-service: &dind-service
      - alias: docker
        name: hub.cloud.ctripcorp.com/devops/docker:20.10.24-dind-trip-0.0.1
        command:
          - --storage-driver=overlay2
          - --insecure-registry=hub.cloud.ctripcorp.com
          - --tls=false
    
    .image-variables: &image-variables
      DOCKER_HOST: tcp://docker:2375
      DOCKER_TLS_CERTDIR: ""
    ```

* 有状态应用申请磁盘后会自动识别路径，应用不需要做改动

* npm install 404 http://conf.ctripcorp.com/pages/viewpage.action?pageId=2569510507

* HPA只扩容不缩容，只扩不缩因为beacon自定义指标qpm取不到值，进入了保护模式，不允许缩容了

* 拉取代码提示blocked，gitlab再登录一下就行了

* 固定IP迁移，新建group即可

* 合并的时候可以直接勾选delete source branch 删分支，代码比较的时候是看commit，合并的时候把commit squashed 了，所以source branch 的commit不在target branch里，此时会被视为未MR

* 时区会影响captain页面显示问题

* captain 堡垒找不到实例是soa实例没注册上

* reset 接口目前不对外 只给NOC

* gitlab的compare version功能 在同一个分支下只可比较增量，即source晚于target，否则会显示空白，显示空白时交换source与target即可

* Not Found! 1、软路由服务无法匹配到该域名，请检查域名指向是否正确； 2、请检查是否成功申请了域名。 重建入口，不行再找zmm

* deploy配置自动合并，Settings-->Integrations-->Maven Deploy-->configure-->Ff check and merge

* artifact 默认过期时间为1周，可以自行在pipeline里修改过期时间

* paas镜像报错unzip: cannot find zipfile directory in one of 100029076-20240711200625-dev_flight_change-3451914.zip or。。。 and cannot find 100029076-282407112625-dev_flight_change-3451914zip.ZIP，period，找肖小龙

* process.env 是 nodejs 取环境变量的方法，需要修改变量时可以直接在CI/CD Variables里修改

* CI/CD 里面可以看到可用的runner tag列表，需要修改runner时可以参考

* node发布无日志，报错path:/api/build.json，需要降级node版本

* Error: Cannot find module 'typescript'，应用问题

* SLB的path是指只能访问服务以path开头的路由，如设置it,则只能访问it开头的路由 无法访问vi，可以建多个入口，以域名和path阻断来控制服务功能的提供

* SLB的member port是点火端口【服务端口】，需要保持一致

* 集群只有一台机器，虽然会先拉起一台机器发布，但堡垒机发布完成后，老机器也就开始销毁了，就导致集群内没有提供服务的机器了，从而出现502。

* SLB下线入口，可以看SLB日志，判断TrafficGroups问题

* job重试500，pipeline重试500，是流水线改过

* captain配置的自动mr用户，默认为审批生产的用户，不是发布的用户

* dump文件不能用zip解压，需要用tar解压，否则分析会报错

* hpa自动纳管会修改最小实例数量，可以找李宗卿&杨培丽申请加白http://conf.ctripcorp.com/pages/viewpage.action?spaceKey=CIS&title=Autopilot-SmartHPA

* 开启atrust会导致DNS解析错误的问题，如连接wifi等操作

* Cache Node Modules是install 的 缓存，如同一个commit 的话， 缓存的key一致， install 的jog就不执行了，cache的 id 不一致就会执行

* gitlab的Administrator用户可以手动邀请到工程中，需要超管邀请，普通owner invite功能禁用了

* 版本冲突，不要用排除，用dependencyManagement，不是parent依赖的优先级高，而是parent的是dependencyManagement，子项目的是直接依赖的间接依赖

* 流水线添加应用报错api error cuz by: failed to parse uri with req:[&{0}]: Key: "GetProjectReq.ID' Error:Field validation for 'lD' failed on the 'required' tag，Group或项目名修改过，找李俊虎

* 


# FAQ

## 请教一个问题，依赖引入了两个版本不一样的包，但是compile却通过了

maven 默认有依赖冲突解决的 ，如果安装的不是低版本 并且低版本没有配置拦截规则的话，compile job 里的 enforcer 检查插件拦不到低版本，拦截规则，申请拦截规则（maven 默认有依赖冲突解决的 ，如果安装的不是低版本 并且低版本没有配置拦截规则的话，compile job 里的 enforcer 检查插件拦不到低版本，拦截规则）

这个报错应该是 maven 默认解决不了版本冲突的问题，本地 mvn compile 试下，想知道生效的是哪个可以搜 maven 依赖冲突解决策略

# 待办事项

webhook在gitlab的使用

自定义job+自定义镜像

策略应对脚本：机器人，conf，历史会话并发，其中conf可手动

