---
title: Linux查询日志
tags:
  - Linux
  - tail命令
categories:
  - 运维与部署
excerpt: 这是文章摘要
thumbnail: https://pan.mwm.moe/f/pvEUk/36.WEBP
cover: https://t.mwm.moe/pc
sticky: 1
date: 2022-03-02 13:09:01
---

# Linux基础命令
## 日志操作
- 查看日志文件倒数1000行:  `tail -fn 1000 {日志文件名称}`
- 服务器日志文件下载: `scp {用户名}@{ip地址}:{服务器文件路径} {本地路径}`
- 日志关键字搜索: 
  - `tail -{最近的行数}f {文件名} | grep "{查询的关键字}" --color=auto`
    eg：`tail -1000f log.log | grep "请求参数" --color=auto`  查询文件log.log的最近1000行中匹配“请求参数”关键字，并将关键字按照红色显示
  - `tail -{最近行数}f {文件名} | grep "{查询的关键字} -A {匹配行之前的行数} -B {匹配行之后的行数} --color=auto"`
    eg: `tail -1000f log.log | grep "请求参数" -A 10 -B 10 --color=auto` 查询log.log文件最近1000行中能够匹配“请求参数”关键字的行并显示匹配行前10行和后10行，匹配的关键字以红色显示
  - `tail -fn {最近行数} {文件名} | grep "{查询的关键字} -A {匹配行之前的行数} -B {匹配行之后的行数} --color=auto"`
    eg: `tail -fn 1000 log.log | grep "请求参数" -A 10 -B 10 --color=auto`  **实时**查询log.log文件最近1000行中能够匹配“请求参数”关键字的行并显示匹配行的前10行和后10行，匹配的关键字以红色显示

