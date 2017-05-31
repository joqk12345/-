---
layout: "post"
title: "使用Docker Compose管理多个容器"
date: "2017-05-31 18:06"
---

## Docker compose
1. Docker compose是一个Docker工具
  2. 可以定义和运行复杂应用的Docker工具
  3. 在文件中定义一个多容器应用
  4. 在文件中使用一条命令来启动你的应用,完成一切准备工作
  5. github.com/docker/compose
2. 一个使用docker容器的应用,通常有多个容器组成
  3. 使用Docker compose ,不再需要使用shell脚本来启动容器
  4. 在配置文件中,所有的容器通过services来定义,然后通过docker-compose脚本来启动,停止/重启应用和应用中的服务以及所有依赖服务的容器
  5. 完整的命令列表如下
  ```
  bulid   构建或重建服务
  help    命令帮助
  kill    杀掉容器
  logs    显示容器的输出内容
  port    打印绑定的开放端口
  ps      显示容器
  pull    拉去服务镜像
  restart 重启服务
  rm    删除停止的容器
  run   运行一个一次性命令
  scale 设置服务的容器数目
  start 开启服务
  stop 停止服务
  up  创建并启动容器
  ```
命令参考
[1]:https://docs.docker.com/compose/install/   "安装"

## 1. 按装Docker Compose
```
curl -L https://github.com/docker/compose/releases/download/1.13.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod  x /usr/local/bin/docker-compose
```
## 2. 配置文件
1. compose的配置文件docker-compose.yml.

```
mysqldb:
image: [classroom.example.com:5000/]mysql
environment:
MYSQL_DATABASE: sample
MYSQL_USER: mysql
MYSQL_PASSWORD: mysql
MYSQL_ROOT_PASSWORD: supersecret
mywildfly:
image: [classroom.example.com:5000|arungupta]/wildfly-mysql-javaee7
links:
- mysqldb:db
ports:
- 8080

```
表明:
1. 定义了两个服务分别为mysqldb和mywildfy
2. 使用image 定义每个服务的镜像名称
3. Mysql容器的环境变量定义在environment
4. Mysql容器使用links和WildFly容器连接
5. 使用ports实现端口转发

## 3. 启动服务
1. 如果从互联网上运行,将docker-compose-internet.yml保存为docker-compose.yml .
2. 如果使用教师给的经镜像,将docker-compose-instructor.yml保存为docker-compose.yml.
3. 使用下面的命令,所有的服务使用后台模式启动
> docker-compose up -d
4. 验证启动服务

```
docker-compose ps
    Name                       Command               State                Ports
attendees_mysqldb_1     /entrypoint.sh mysqld            Up      3306/tcp
attendees_mywildfly_1   /opt/jboss/wildfly/customi ...   Up      0.0.0.0:32773->8080/tcp
```
这里提供了一个整合的列表显示所有启动的服务和容器。
通常使用docker ps 命令来验证应用的容器,和在Docker诸己上运行的其他容器.
5. 查询服务日志

```
taching to attendees_mywildfly_1, attendees_mysqldb_1
mywildfly_1 | => Starting WildFly server
mywildfly_1 | => Waiting for the server to boot
mywildfly_1 | ===========================================================
mywildfly_1 |
mywildfly_1 |   JBoss Bootstrap Environment
mywildfly_1 |
mywildfly_1 |   JBOSS_HOME: /opt/jboss/wildfly
mywildfly_1 |
mywildfly_1 |   JAVA: /usr/lib/jvm/java/bin/java
mywildfly_1 |
mywildfly_1 |   JAVA_OPTS:  -server -Xms64m -Xmx512m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true
mywildfly_1 |
. . .
mywildfly_1 | 15:40:20,866 INFO  [org.jboss.resteasy.spi.ResteasyDeployment] (MSC service thread 1-2) Deploying javax.ws.rs.core.Application: class org.javaee7.samples.employees.MyApplication
mywildfly_1 | 15:40:20,914 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) JBAS017534: Registered web context: /employees
mywildfly_1 | 15:40:21,032 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 28) JBAS018559: Deployed "employees.war" (runtime-name : "employees.war")
mywildfly_1 | 15:40:21,077 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
mywildfly_1 | 15:40:21,077 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
mywildfly_1 | 15:40:21,077 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.2.0.Final "Tweek" started in 9572ms - Started 280 of 334 services (92 services are lazy, passive or on-demand)
mysqldb_1   | Running mysql_install_db
mysqldb_1   | 2015-06-05 15:38:31 0 [Note] /usr/sbin/mysqld (mysqld 5.6.25) starting as process 27 ...
mysqldb_1   | 2015-06-05 15:38:31 27 [Note] InnoDB: Using atomics to ref count buffer pool pages
. . .
mysqldb_1   | 2015-06-05 15:38:40 1 [Note] Event Scheduler: Loaded 0 events
mysqldb_1   | 2015-06-05 15:38:40 1 [Note] mysqld: ready for connections.
mysqldb_1   | Version: '5.6.25'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
mysqldb_1   | 2015-06-05 15:40:18 1 [Warning] IP address '172.17.0.24' could not be resol
```
## 4. 验证应用
通过 http://dockerhost:32773/employ访问应用,在浏览器显示如下:
## 5. 扩展服务
你能想象这样扩展服务:

```
docker-compose scale mywildfly=4
Creating and starting 2... done
Creating and starting 3... done
Creating and starting 4... done
```
检查日志:
> docker-compose logs
检查运行的实例:
> docker-compose ps

你也能减少运行的实例数目:

```
docker-compose scale mywildfly=2
Stopping rafael_mywildfly_3... done
Stopping rafael_mywildfly_4... done
Removing rafael_mywildfly_3... done
Removing rafael_mywildfly_4... done

```

## 6. 停止服务
停止服务:

```
docker-compose stop
Stopping attendees_mywildfly_1...
Stopping attendees_mywildfly_2...
Stopping attendees_mysqldb_1...
```
