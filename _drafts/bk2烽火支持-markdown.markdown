---
layout: "post"
title: "bk2烽火支持.markdown"
date: "2017-05-26 18:52"
---

1. hadoop fs -ls /
2. hadoop fs -ls /hbase1


## 错误的解决方式

```
hmaster 闪退,无法连接到hbase
hbase file layout needs to be upgraded  {hbase 中的文件损坏}
```

### 解决方式
1. 不使用自带的zookeeper,用独立的zookeeper
  2. zookeeper的连接配置一个不常用的连接比如2222
    1. 你在hbase-site.xml中添加一个配置

    ```
      <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2222</value>
      </property>
      ```
    2. 修改hbase-env.sh 配置文件
    > hbase-env.sh --> export HBASE_MANAGES_ZK=false
3. 检查hadoop的权限,将hadoop中的文件权限赋予777
  3. 退出安全模式
    > hadoop dfsadmin -safemode leave
  4. 查看文件权限
    > hadoop fs -ls /hbase

  ```
drwxr-xr-x   - root supergroup          0 2017-05-26 18:57 /hbase1/.tmp
drwxr-xr-x   - root supergroup          0 2017-05-26 18:57 /hbase1/MasterProcWALs
drwxr-xr-x   - root supergroup          0 2017-05-26 18:57 /hbase1/WALs
drwxr-xr-x   - root supergroup          0 2017-05-25 12:09 /hbase1/archive
drwxr-xr-x   - root supergroup          0 2017-05-23 19:53 /hbase1/data
-rw-r--r--   1 root supergroup         42 2017-05-23 19:53 /hbase1/hbase.id
-rw-r--r--   1 root supergroup          7 2017-05-23 19:53 /hbase1/hbase.version
drwxr-xr-x   - root supergroup          0 2017-05-26 20:08 /hbase1/oldWALs
  ```
错误启动因为缺少hbase.version /hbase.id /MasterProcWALs

  5. 修改文件权限
    > hadoop fs -chmod 777 /hbase


等待退出安全模式
> bin/hadoop dfsadmin -safemode wait
> 检查hdfs的健康状态
> bin/hadoop fsck /

> bin/hadoop fs -cp /hbase/hbase.version /hbase.bk/
> bin/hadoop fs -rmr /hbase
> bin/hadoop fs -mv /hbase.bk /hbase
