---
title: MySQL日志
tags:
  - MySQL
  - Bin Log
  - Redo Log
  - 日志
categories:
  - MySQL
excerpt: MySQL的bin log和redo log学习记录
thumbnail: https://t.mwm.moe/fj/
cover: https://t.mwm.moe/pc
sticky: 1
date: 2023-08-13 16:31:07
---

MySQL的日志种类
----------

*   **错误日志（Error Log）：** 记录MySQL服务器运行过程中发生的错误和警告消息。错误日志对于排查和解决数据库问题非常有帮助。
*   **查询日志（Query Log）：** 记录所有进入MySQL服务器的查询语句，可以用于跟踪和分析数据库的查询操作。
*   **二进制日志（Binary Log）：** 记录对数据库的更改操作，包括INSERT、UPDATE、DELETE等。这些日志用于数据恢复、复制和故障恢复。
*   **慢查询日志（Slow Query Log）：** 记录执行时间超过指定阈值的查询语句，帮助识别并优化性能较差的查询。
*   **中继日志（Relay Log）：** 在主从复制中，从服务器上的中继日志记录了主服务器上的二进制日志的内容，用于从服务器的数据复制。
*   **错误交换日志（Error Log）：** 用于MySQL复制，记录复制中的错误信息和事件。
*   **事务日志（Transaction Log）：** 用于事务的持久性和数据恢复，通常在InnoDB存储引擎中使用，包括redo log重做日志和undo log回滚日志。
*   **日志文件（General Log）：** 记录所有连接到MySQL服务器的客户端请求，包括查询、连接和断开连接等。

数据库的两阶段提交
---------

为了解决MySQL程序异常时主从库的数据不一致问题。

> 主库通过redo log恢复，从库通过bin log同步数据。 如果不使用两阶段提交，因为redo log日志和bin log日志的写入点不同，可能导致redo log记录下的事务，而bin log没有记录，因此主库恢复时有该事务，而从库是根据bin log同步的因此从库没有该事务，so主从库的数据不一致。

### 两阶段提交流程图

![两阶段提交](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230813153035.png)

### 数据库恢复数据流程

![数据恢复](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230813153520.png)

重做日志(Redo Log)
--------------

> redo log主要保证数据的持久性和完整性

### 使用redo log原因

*   数据页大小是16KB，而redo log仅几Byte，数据量小
*   数据页刷盘是随机写，redo log是顺序写，比较块
*   此上两条，redo log刷盘比数据也直街刷盘的并发能力高

### redo log刷盘流程

1.  事务中修改数据
2.  写入redo log 到Log Buffer中
3.  将Log Buffer 写入文件系统缓存(page cache)中
4.  将page cache写入磁盘中

刷盘时机
----

*   事务提交时，根据`innodb_flush_log_at_trx_commit` 判断是否刷盘
    *   0：事务提交时不进行刷盘操作
    *   1：每次事务提交都进行刷盘操作
    *   2：将日志写入到page cache
*   log buffer空间不够时(大概占据总容量的一半)，将日志刷盘
*   事务日志缓冲区满进行刷盘
*   检查点(check point)操作刷盘
*   后台刷新线程，InnoDB后台线程每一秒将脏页刷新到磁盘中，同时redo log也会算盘
*   关闭服务器时进行刷盘

### 日志文件组

磁盘上的redo log采用环形数组形式，具有两个指针checkpoint和write pos。当redo log写入数据达到 write pos 环绕一圈追上checkpoint时，MySQL会清空一些记录，将checkpoint向前推进.

二进制日志(Bin Log)
--------------

> 数据库集群进行主从复制等操作可以通过bin log保证数据的一致性

### 主从复制流程

1.  主数据库将数据的变化写入到bin log
2.  从库连接主库
3.  从库创建I/O线程向主库请求更新bin log
4.  主库创建binlog dump线程并向从库发送bin log，从库通过I/O线程进行接收
5.  从库I/O线程将接收到的bin log写入relay log中
6.  从库SQL线程读取relay log并同步数据

### 记录格式

*   statement：直接记录sql语句，容易出现数据不一致问题，eg：create\_time=now()
*   row: 记录sql语句和具体的操作数据，eg：create\_time=具体的时间戳
*   mixed: 当不会造成数据不一致时使用statement方式，否则使用row方式

> statement会造成数据不一致，row会占用更多的空间，消耗更多的IO资源，因此使用mixed

### 刷盘流程

1.  事务修改数据
2.  将bin log写入binlog cache
3.  binlog cache写入到文件系统缓存(page cache)
4.  将page cache写入到磁盘中

### 刷盘时机

根据`sync_binlog`控制

*   0：由系统判断何时写入磁盘
*   1： 每次提交事务时刷盘
*   N: 每N个事务后写入磁盘

### 生成新的binlog文件

*   MySQL停止或重启
*   使用`flush logs`命令
*   binlog大小超出`max_binlog_size`