---
layout: "post"
title: "what is docker"
date: "2017-06-01 10:57"
---
## 什么是docker?

过去一年,我在docker上发表了一组博客.我没有完全相信我已经掌握和理解或者解释了docker是什么.
这一次是另一个尝试去完全理解和解释docker的基本与原则和虚拟化的发展.

## 1. pre-virtualization{未虚拟化前夜}

在虚拟化来临之前,服务器是有一个OS一台机器跑一个应用.在一台物理机上跑多应用经常会导致资源冲突.最安全的做法是在一台机器上跑一个应用.导致的结果,数量昂贵,服务会增倍,但是不能提供比例值作为回报.一些评估中,88%服务资源都处于没有充分利用.这导致了在服务器虚拟化成了一片肥沃的市场去开发以此降低基础设施花费并且增长的灵活性.

![未虚拟化](https://indrabasak.files.wordpress.com/2017/03/conventional-os.png?w=300&h=269)

## 2. Virtualization{虚拟化阶段}

虚拟化技术引入了一个叫做virtual machine(VM)的概念,通过在一个物理机上创建多个执行环境,给用户一种真实机器的错觉.例如,创建拥有自己资源(CPU.RAM,etc),VMs在一个物理机下保持每个虚拟机隔离.优势就是
1. 可以在一个物理机环境下跑多个虚拟机,
2. 更容易在不同物理之间进行迁移应用,来减少最小的宕机时间
3. 降级拥有成本
4. 方便进行程序跑在不兼容的os上进行调试

为了达到此目的,我们引入了一个必要的层在物理机栈里面,叫做virtualization layer.虚拟化的层级可以在硬件层/操作系统层以及应用层.
![虚拟化层级](https://indrabasak.files.wordpress.com/2017/04/virtualization-types.png?w=703&h=300)

1. 硬件层虚拟化
  2. 基于本地cpu来做的基于gest OS的仿真指令
2. os层的虚拟化
  3. VM和host在OS 层,他重定向了来自vm的I/O和系统请求到host OS.
3. 应用层虚拟化
  4. 类似于java虚拟机中断二级制代码到本地code

最好好的隔离是虚拟化层离硬件最近.然而,他导致了花费更高的资源需求和灵活性.
然而隔离在VMs和物理 OS 导致了虚拟化层移出了机器stack,他增加了灵活性和系能.


```
在硬件层的虚拟化就如Type-1 Hypervisors,虚拟化层跑在裸金属上.不需要任何的
Host OS.VMs作为用户进程在用户模式下跑,除此之外,VM OSs知道虚拟机内核模式.
当gest OS执行了敏感的指令模式,虚拟层中断指令,并且检查他的准备情况.如果指令在
gestOS 傻姑娘做好了准备,虚拟化层立刻执行此指令.如果指令来自vm的用户程中,他模拟了内核行为或者硬件
```

```
操作系统层的虚拟化,虚拟化层跑在host OS上面,这里有不同的方式关于虚拟化层与Vm侧沟层交互.
```
1. full vm(完全vm)
  2. 在一个 full featured OS-level virtualization,一个VM有一个完全独立于host OS的自己的gestOS,类似于type-2
   Hypervisors.
  3. gest OS 不需要修改
  4. 在gestOS loading时内核指令会被动态替换
  5. 与此通知用户侧沟男的指令直接被跑在host OS/processor层的virtualization层执行
2. OS Container(os 容器虚拟化)
  3. os Container是另一种类型的OS-level virtualization.
  4. 多个vm共享一个单一的host OS kernel
  5. VMs共享host的执行环境
  6. 没有任何权限去修改他们
  7. VM的任何改变包括状态的改变,都首先与vm's本地环境
  8. 这让 OS containers 有耕地的资源需求和更高的扩展性
![OS container](https://indrabasak.files.wordpress.com/2017/04/os-level-vir-redirect2.png?w=461&h=292)

 OS container 限制从ageVM 到一个VM's local resoure partion的访问请求
 这种重定向传输VMs跟linux中的chroot挺相似的.共享读资源,直接通过VMs访问不需要copied.

OSContainer的一个例子就是LXC(linux Containers).LXC引入了一组概念例如cgroups(control groups)允许局限和优先访问host 资源(CPU/memory/etc),命名空间隔离等.

  ![](https://indrabasak.files.wordpress.com/2017/03/os-level-virtualization.png?w=300&h=292)

Docker最初使用的是LXC,但是后来使用了Open Containers Initiative(OCI)容器标准,避免在容器生态的碎片化.一个Docker容器潜在会被一个OCI替代,例如windows Server Containers.

## vm和docker的不同之处?

full vm使用的是host OS,没有共享内核,这允许full VM 在一台物理机上面跑不同的os.
docker 共享host OS kernel 导致他跑的容器跟它的容器是一样的.也就是你能创建或者运行镜像基于不同的linux发行版由于他们共享同样的内核.
1. 优点
  2. 由于docker花费了很少的资源,你可以在一个物理机上面跑很多的容器,降低拥有成本,full vm 有自己的资源和最小的共享
  3. Docker 容器共享通用的读部分通过分层的文件系统,1000个实例每个实例1GB的docker容器的镜像花费的空间是少于1Tb的.
  4. 快速启动时间是按照秒级的超过fullvm的分钟级别的
  5. 容易部署
2. 劣势
  3. 缺少隔离
  4. 不能跑在不同的os

![docker Container](https://indrabasak.files.wordpress.com/2017/04/docker-vs-hypervisor.png?w=683&h=341)

##　Docker的架构

我们分析一下docker的主要组件:
1. Docker Engine
  2. docker的主要入口,负责
    3. 构建docker image
    4. 编排
    5. 管理卷积
    6. 网络
    7. 规模迁移
2. Docker Command Line(CL)
  3. 命令行工具与Docker进行交互
  4. 一旦你请求docker Engine 去运行一个image,他委派containerd进程负.
    5. containerd下载并且存储改容器的镜像;
    6. 创建根文件系统
    7. 通过container-shim去调用runC来启动container
    8. 监控容器
    9. 管理低级别的存储
    10. 管理网络接口
![](https://indrabasak.files.wordpress.com/2017/03/docker.png?w=602&h=386)

1. gPRC
  2. 在containerd和DockerEngine 之间的通信协议
2. runC是一个进程负责
    4. 创建和启动容器
    5. 他构建在libcontainer
    6. 一旦runC从container-shim,接收image和配置
    7. 他就会启动一个container作为一个子进程
    8. 一旦容器启动,他负责管理container到container-shim
    9. 配置部分包含需要的信息去创建一个容器
3. container-shim
  4. 在容器退出runC,后管理无头的containers
4. Docker container
  5. 基于Open Containers Initiative(OCI)
  6. 在一个容器里,就是一个隔离环境不影响OS的资源(CPU/memory/disk/networkt)
  7. 又有一个最小的OS(除了kernel/system call inteface/drivers/etc)必要去跑一个应用
  8. 使用a layered filesystem叫做AuFS.包括读和写部分.
    9. 读的部分共享不同的容器
    10. 写通过隔离的挂载来实现
5. Docker Image
  6. building block for application
  7. 是个tar archive包含所有的文件/库/二进制/应用所需的机器指令
  8. 被构建在一个层上面.
6. docker-example
7. running an Application in Docker
![](https://indrabasak.files.wordpress.com/2017/03/docker-deployment1.png?w=587&h=453)

[参考文档][fb507b05]

  [fb507b05]: https://indrabasak.wordpress.com/2017/04/03/what-is-docker/ "参考文档"
