---
layout: post
title: "linux定时任务"
description: ""
category: linux
tags: []
---
> 在linux系统中，经常会定期的运行一些脚本，来维护网站。比如说，在晚上数据访问量小的时候，对数据进行备份，或者需要将数据同步到数据库的操作等，都需要用到定时任务。linux系统提供了两种定时任务，一种是只会执行一次的 at计划任务，另一种是周期执行的 cron计划任务。

### at 一次计划任务

---
在使用at计划任务前需要确保atd服务是否开启，否则命令不会被执行。可以使用service atd start 开启。

```
用法：at  时间
选项：
    -m      当计划任务结束时，发邮件给用户。
    -l      查看用户的at 计划任务。
    -d      删除用户的at 计划任务。
    -c      查看at 计划任务的具体内容。
demo ：
    at 17:00
    shotdown -h now
    #可以输入多条命令后，按ctrl+D快捷键结束。

    at -l   查看at 计划任务。
    at -d   删除at 计划任务。
```
ps : at 有许多时间格式，小时:分钟(默认代表当天时间)，at 4pm + 3days(代表3天后的下午4点执行)，at 14:30 2014-12-30(指定年月日以及日期的计划任务)。

---

### cron 周期性计划任务

---
在使用crontab计划任务前需要确保crond服务是否开启，否则命令不会被执行。可以使用service atd start 开启。可以使用chkconfig crond on是否开机启动。

```
作用：为每个用户维护周期性的计划任务文件。
用法：crontab [-u 用户] [-l|-e|-r]
选项：
    -u      指定计划任务的用户，默认当前用户。
    -l      查看用户的cron 计划任务。
    -r      删除用户的cron 计划任务。
    -e      编辑 cron 计划任务。

时间设置：
    第一段：代表分钟 0—59
    第二段：代表小时 0—23
    第三段：代表日期 1—31
    第四段：代表月份 1—12
    第五段：代表星期几，0代表星期日 0—6
    第六段：命令

demo ：
    crontab -e
    59 23 * * * tar -jxvf log.tar.gzip /var/log     #每天晚上23:59分钟进行日志备份
    00 */3 * * * who                                #每3小时整点检查用户登录情况 
    0 0 1,15 * * fsck /home                         #每月1号和15号检查/home 磁盘

    crontab -u dnzhu -e         #指定cron的用户
    crontab -l                  #查看当前用户创建的cron。
    crontab -r                  #删除当前用户的cron。

```
ps : 为了控制用户随意定义自己的计划任务，管理员可以进行ACL访问控制，at计划任务的控制文件分别为/etc/at.allow和/etc/deny。cron计划任务的控制文件分别是/etc/cron.allow和/etc/cron.deny,默认allow文件是不存在的。在控制文件中，写入用户名即可，格式一行一个用户名。

---

