---
title: 线上CPU飙高解决方案
tags:
  - 线上问题
categories:
  - 线上问题
excerpt: 线上CPU升高的原因和排查方向, 以及部分查看排查时查看信息的命令
thumbnail: https://cdn.cdnjson.com/tvax3.sinaimg.cn//large/0072Vf1pgy1foxkd8fiqgj31hc0u0h3v.jpg
cover: https://t.mwm.moe/pc
sticky: 1
date: 2022-08-17 21:32:01
---

## 相关命令
1. 查看cpu占用最大的进程：`top -c`
2. 查看进程下线程的cpu: `top -Hp {进程号(PID)}`
3. 获取线程号: `printf "%x\n" {线程的PID}`
	> 如果输出db1, 则线程号为 0xdb1
4. 获取线程的状态和信息: `jstack {进程号} | grep {线程号(例如0xdb1)}`
5. 查看线程异常信息: `jstack -l {线程PID}`, 也可以保存到文件中查看: `jstack -l {线程PID} > {文件名}.stack`
6. 获取进程持续时间的GC情况: `jstat -gcutil {进程号PID} {统计间隔(毫秒)} {统计次数}`
	> `jmap -heap {进程PID}` 查看进程堆内是否要溢出了
7.  导出进程的dump文件
	> `jmap -dump:format=b,file={文件名}.dmp {进程PID}` 体积和堆一样大，速度慢
	> `jmap -dump:live,format=b,file={文件名}.hprof {进程PID}`，堆内存活的dump文件，体积小于堆，hprof可以给MAT分析用（推荐）
8. Linux查看Java进程的启动参数(主要用来查看堆空间分配，和GC方案的采用)
	- `ps eww {进程PID}`
	- `jcmd {进程PID} VM.flags`
	- `jinfo -flags {进程PID}`
	- `jmap -heap {进程PID}`

## 原因
1. 内存消耗过大, 导致Full GC过多
2. 代码中有大量消耗CPU的操作，导致CPU过高，系统运行缓慢
3. 由于锁使用不当，导致死锁
4. 随机出现大量线程访问接口，导致缓慢
5. 某个线程由于某种原因一直在WAITING状态，导致此时该功能不可用

## 方向
1. 查看监控平台，看http请求和feign请求是否异常
2. 查看满接口平台，查看时间较长的请求接口
3. 查看数据库访问，找到查询数据量较大的sql或慢sql
4. 查看JVM监控，查看堆内存的异常或GC异常情况
5. 查看线程是否有异常情况
