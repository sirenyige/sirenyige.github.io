---
title: redoLog和binLog日志
date: 2020-07-05 23:58:26
categories:
tags:
---

# redoLog和binLog日志

<!-- more -->

## 一、redo log（重做日志）

如果每一次的数据库更新操作都需要写进磁盘，整个过程 IO 成本、查找成本都很高。为了解决这个问题，MySQL 使用了 WAL 技术，WAL 的全称是 Write-Ahead Logging，它的关键点就是先写日志，再写磁盘。

在同一个事务中，每当数据库进行修改数据操作时将修改结果更新到内存后，会在redo log添加一行记录记录“需要在哪个数据页上做什么修改”，并将该记录状态置为prepare，等到commit提交事务后，会将此次事务中在redo log添加的记录的状态都置为commit状态，之后将修改落盘时，会将redo log中状态为commit的记录的修改都写入磁盘。

InnoDB 的 redo log 是固定大小的循环存储，从头开始写，写到末尾就又回到开头循环写，当写满时，也会进行操作的更新以擦除redo log中的一些记录。

有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 **crash-safe**。

### 1、作用：

确保事务的持久性。防止在发生故障的时间点，尚有数据未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

### 2、内容：

物理格式的日志，记录的是物理数据页面的修改的信息。

### 3、产生时机：

事务开始之后就产生redo log，redo log的落盘并不是随着事务的提交才写入的，而是在事务的执行过程中，便开始写入。

### 4、释放时机：

当对应事务的数据写入到磁盘之后，redo log的使命也就完成了，日志占用的空间就可以重用（被覆盖）。

### 5、redo log写文件流程

redo log包括两部分：一是内存中的日志缓冲(redo log buffer)，该部分日志是易失性的；二是磁盘上的重做日志文件(redo log file)，该部分日志是持久的。

MySQL支持用户自定义在commit时如何将log buffer中的日志刷log file中。这种控制通过变量innodb_flush_log_at_trx_commit 的值来决定。该变量有3种值：0、1、2，默认为1。但注意，这个变量只是控制commit动作是否刷新log buffer到磁盘。

- 当设置为1的时候，事务每次提交都会将log buffer中的日志写入os buffer并调用fsync()刷到log file on disk中。这种方式即使系统崩溃也不会丢失任何数据，但是因为每次提交都写入磁盘，IO的性能较差。
- 当设置为0的时候，事务提交时不会将log buffer中日志写入到os buffer，而是每秒写入os buffer并调用fsync()写入到log file on disk中。也就是说设置为0时是(大约)每秒刷新写入到磁盘中的，当系统崩溃，会丢失1秒钟的数据。
- 当设置为2的时候，每次提交都仅写入到os buffer，然后是每秒调用fsync()将os buffer中的日志写入到log file on disk。



## 二、binlog（归档日志）

binlog是Mysql sever层维护的一种二进制日志，与innodb引擎中的redo/undo log是完全不同的日志；其主要是用来记录对mysql数据更新或潜在发生更新的SQL语句，并以"事务"的形式保存在磁盘中；

### 1、作用

- 复制：MySQL Replication在Master端开启binlog，Master把它的二进制日志传递给slaves并回放来达到master-slave数据一致的目的
- 基于时间点的数据恢复
- 增量备份

### 2、内容：

逻辑格式的日志，可以简单认为就是执行过的事务中的sql语句。

### 3、产生时机：

事务提交的时候，一次性将事务中的sql语句（一个事物可能对应多个sql语句）按照一定的格式记录到binlog中。

### 4、释放时机：

binlog的默认是保持时间由参数expire_logs_days配置，也就是说对于非活动的日志文件，在生成时间超过expire_logs_days配置的天数之后，会被自动删除。

## 三、数据更新提交流程

![更新流程](/redoLog和binLog日志/更新流程.png)

## 四、事务恢复

<img src="/redoLog和binLog日志/事务恢复.png" alt="事务恢复" style="zoom: 67%;" />

在MySQL启动后
1、初始化存储引擎，如本例中的InnoDB引擎
2、InnoDB引擎层读取redolog进行InnoDB层的故障恢复，回滚未prepared和commit的事务，但对于已经prepared，但未commit的事务，暂时挂起，保存到一个链表中，等待后续读取binlog日志，根据binlog日志再对这部分prepared的事务进行处理。
3、接下来，MySQL会读取最后一个binlog文件。通过文件头上是否存在标记LOG_EVENT_BINLOG_IN_USE_F，判定上次MySQL是正常关闭还是异常关闭，如果是异常关闭，则会进入故障恢复过程。 
4、进入故障恢复过程后，会依次读取最后一个binlog文件中的所有log event，并将所有已提交事务的binlog日志中记录的xid提取出来添加到hash表中，以备后续对前述InnoDB故障恢复后遗留的Prepared事务继续处理。另外此处还要定位最后一个完整事务的位置，防止在上次系统异常关闭时有部分binlog日志未刷到磁盘上，即存在写了一半的binlog事务日志，这部分写了一半binlog日志的事务在MySQL中会按事务未提交来处理，后续会将其在存储引擎层回滚。**当此文件中的内容全部读出之后，一是得到一个已提交事务的列表，另一个是最后一个完整事务的位置。** 
5、然后检查由InnoDB层得到的Prepared事务列表，若Prepared事务在从Binlog中得到的提交事务列表中，则在InnoDB层提交此事务，否则回滚此事务。
6、最后MySQL将最后一个完整事务位置之后的binlog清除，完成故障恢复全部过程。