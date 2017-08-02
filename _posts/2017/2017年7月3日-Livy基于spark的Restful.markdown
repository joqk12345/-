
##　Livy一个基于Apache Spark的Rest服务

1. 以Rest的方式代替了Spark传统的处理交互方式
2. 提供了多用户,安全/以及容错的支持.

### 两种方式处理数据
1. 交互式处理
    1. 用户使用spark-shell或是pyspark脚本启动spark应用程序
    2. Spark会在当前终端启动REPL(Read-Eval-Print Loop)来接收用户的代码输入,并且将其编译成spark作业提交到集群上去执行,

2. 批处理
    1. 批处理的程序逻辑由用户实现并编译打包成jar
    2. spark-submit 脚本启动Spark应用程序来执行用户所编写的逻辑
    3. 在执行过程中用户没有雨Spark进行任何的交互

3. 问题
    1. 都需要用户登录到Gateway节点上,通过脚本启动Spark进程.
    2. 将资源的使用和故障发生的可能性集中到Gateway节点.所有的Spark进程都是在GateWay节点上启动,这势必会增加GateWay节点的资源使用负担和故障发生的可能性,同时Gateway节点的故障会带来单点问题,造成Spark程序的失败.
    3. 其次难以管理,审计以及与已有权限管理工具的集成.由于Spark采用脚本的方式启动应用程序,因此相比于Web发给弄好吃少了许多管理/审计的便利性,同时也难以与已有的工具结合,如Apache Knox.
    4. 同时也将GateWay节点上的部署细节以及配置不可便面地暴露给登录用户
    5. 位了避免上述这些问题,同时提供原生Spark已有的处理交互方式,并且为Spark带来器所缺的企业级管理,部署和审计功能.我们引入:基于Spark的Rest服务:Livy.
## Livy
    Livy是一个基于Spark的开源Rest服务,他能够通过Rest方式将代码片段或是序列化的二进制代码提交到Spark集群中去执行.他提供了一些这些基本功能:

1.  提交Scala/python或是R代码片段到远端Spark集群上执行;
2.  提交java /scala/python所编写的Spark作业到远端的Spark集群上执行;
3. 提交批处理应用在集群中执行.

> 从livy所提供的基本功能可以看到Livy涵盖了原生Spark所提供的两种处理交互方式.与原生Spark不同的是,所有操作都是通过Rest的方式提交到Livy服务器傻姑娘,再有Livy服务器端发送到不同的Spark集群去执行,我们首先了解一下Livy的架构:

![Livy的基本架构](http://ipad-cms.csdn.net/cms/attachment/201706/5934d153b0cf4.jpg)

1. 接受并解析用户的Rest请求,转换成相应的操作
2. 管理者用户所启动的所有Spark集群.


处理过程,用户可以以REST请求发给弄好吃通过Livy启动一个新的Spark集群,Livy将每个启动的Spark集群称之为一个会话(Session),一个会话是由一个完整的Spark集群所构成的.
1. 交互式会话(interactive session)
    1. Live交互式会话则是在远端的Spark集群中启动REPL,所有的代码/数据都要通过网络来传输.
    2. 创建交互式会话
    3. 提交代码
    4. 查询执行结果
    5. Livy的API设计为阻塞发给弄好吃,当提交代码请求后Livy会立即反抗是请求id,而并非阻塞在该次请求上知道执行完成,因此用户可以通过该id来反复轮训结果,当然只有当该端代码执行完毕之后用户的查询请求才能得到正确的结果.
2. 批处理会话(batch session)
    1. className 和 file 来提交处理任务

## 企业级特性
1. 多用户支持
    1. 采用hadoop中的代理用户(proxy user)模式,代理模式广泛使用与多用户的环境,如HiverServer2.在此模式中超级用户可以代理成普通用户去访问资源,并拥有普通用户相应的权限.开启了代理用户模式后,以用户tom所创建的会话所启动的spark集群用户就会是tom.
    2. ![livy多用户支持](http://ipad-cms.csdn.net/cms/attachment/201706/5934d4f3d3b21.jpg)
    3. 为了使用此功能用户需要配置"livy.impersonation.enabled",同时要在Hadoop中将Livy服务端进程的用户配置为Hadoop proxy .
2. 安全性
    1. 端到端的安全
    2. 客户端认知
        1. 采用基于Kerberos的Spnego认证
        2. Livy服务器配置Spngego认证之后,用户发起Http请求之前必须获得Kerberos认证,只有通过认证之后才能正确访问Livy服务端,不然livy服务端会返回401错误.
        3. Https/SSL
            需要开启SSL加密HTTP协议,以确保传输Http报文的安全.
    3. SASL RPC
        1. 胡天宇SASL认证的RPC通信机制:当livy服务器启动Spark集群时会产生摔成额随机字符串用作两者之间的认证秘钥,只有Livy服务端和该Spark集群之间采用相同的秘钥,这样就保证了只有Livy副刷端和该Spark集群进行通信,防止匿名的连接试图与Spark集群通信.
        2. ![三种安全机制](http://ipad-cms.csdn.net/cms/attachment/201706/5934d538ad4de.jpg)
        这样就构成了Livy完整的端到端的安全机制,确保没有经过认证的用户,匿名连接的用户无法与Livy服务中的任何一个环节进行通信.
3. 失败恢复
    由于Livy服务器端是单点,所有的操作都需要通过Livy转发到Spark集群中,如何确保Livy服务端失效的时候已创建的所有会话不受影响,同时Livy服务端恢复过来后能够与已有的会话重新连接以继续使用?

    Livy 提供了失败恢复机制,当用户启动会话的同时,Livy会在可靠的存储上记录会话相关的元信息,一旦Livy从失败中恢复过来,他会试图读取相关的元信息并与Spark集群重新连接.为了是用该特性我们需要配置Livy使其开启此功能:
    
    失败恢复能够有效地避免因Livy服务端单点故障造成的所有会话的不可用,同时也避免了因Livy服务端重启造成的会话不必要的失效.

## 总结
从spark处理交互方式的局限引出了Livy一个基于Spark的REST服务,同时全面介绍了其基本架构/核心功能以及企业级特性,Livy不仅涵盖了Spark所提供的所有处理交互方式,同时又结合了多种的企业级特性,虽然Livy项目潜在还处于早期,许多的功能有待增加和改进,我相信假以时日,Livy必定成为一个优秀的基于Spark的REST服务.

[文章引用](http://geek.csdn.net/news/detail/208943)

[文章引用](https://zh.hortonworks.com/blog/livy-a-rest-interface-for-apache-spark/)



