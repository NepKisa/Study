# HAflume

使用zookeeper实现flumd的HA，需要自定义一个HAflume服务（内部调用flume的main方法即可整合flume），某一节点挂掉时，leader节点会远程登录服务器执行服务启动命令，需要实现配置服务器之间免密

- 通过配置文件（flumeserver.xml）提供一些参数，参数包含HAflume服务启停的命令
- main方法里初始化一些必要信息，如new zookeeper()，注册zookeeper服务
- 注册zk节点完成后，最后启动一个死循环保持进程存活
- 注册zk节点时，新起一个线程异步调用flume的Application.main()方法，启动flume进程

## tcp通信机制
org.apache.zookeeper.ZooKeeper#ZooKeeper 构造方法会调用cnxn.start()启动发包线程

org.apache.zookeeper.ClientCnxn#start

这个方法会启动两个线程，调用方法(这两个方法均继承了Thread)
- sendThread.start()
- eventThread.start()

> Thread的start方法会调用native的start0方法，start0是jvm中的一个方法，会新启动一个线程来执行run方法

org.apache.zookeeper.ClientCnxn.SendThread#run中会调用
- sendPing()  添加packet到outgoingQueue队列
- clientCnxnSocket.doTransport(....,outgoingQueue..) 内部调用doIO 发送队列中的包

org.apache.zookeeper.ClientCnxnSocketNIO#
doIO()中调用
- sock.write(p.bb) 发送包到服务端

## Watcher回调机制

**回调方法为watcher实现类中的process方法**


- 注册到服务端：客户端注册   Watcher  客户端通过调用 ZooKeeper 的 API（如   getData  、  getChildren   或   exists  ）时，将一个实现了   Watcher   接口的对象传递给 ZooKeeper 服务端。服务端会将这些   Watcher   存储在   WatchManager   中。
- 通知客户端：服务端触发   Watcher  当 ZooKeeper 服务端检测到某个节点的状态发生变化（如数据更新、节点创建或删除等），它会触发与该节点相关的   Watcher  。服务端会创建一个   WatchedEvent   对象，包含事件类型、节点路径等信息，并将其发送给客户端。
- 客户端构造事件队列：客户端接收通知客户端接收到服务端发送的通知后，会将字节流反序列化为   WatcherEvent   对象。然后，客户端会从本地的   ZKWatchManager   中查询与该事件相关的   Watcher  ，并将其存储到   waitingEvents   队列中。
- 客户端回调 process 方法：客户端的   EventThread   线程会不断while循环，轮询   waitingEvents   队列。当队列中有事件时，  EventThread   会从队列中取出事件，并遍历与该事件相关的   Watcher   集合。对于每个   Watcher  ，调用其   process(WatchedEvent event)   方法来执行回调逻辑。

注意事项：
- 一次性触发：每个   Watcher   只能触发一次，触发后需要重新注册。队列里调用的take方法会移除元素
- 异步处理：  Watcher   的回调是在独立的线程中执行的，不会阻塞主流程。通过以上机制，ZooKeeper 的服务端可以有效地回调客户端的   Watcher   的   process   方法，实现对节点状态变化的实时响应。


