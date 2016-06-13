title: supervisor 教程
date: 2015-10-19 01:48:46
tags: Supervisor Linux
---
Supervisor是一个进程监控程序。

 

满足的需求是：我现在有一个进程需要每时每刻不断的跑，但是这个进程又有可能由于各种原因有可能中断。当进程中断的时候我希望能自动重新启动它，此时，我就需要使用到了Supervisor

 

**先弄懂两个命令**:

supervisord : supervisor的服务器端部分，启动supervisor就是运行这个命令

supervisorctl：启动supervisor的命令行窗口。

需求：redis-server这个进程是运行redis的服务。我们要求这个服务能在意外停止后自动重启。

**安装（Centos）**：

yum install python-setuptools

easy_install supervisor

 

**测试是否安装成功**：

echo_supervisord_conf

 

**创建配置文件**：

echo_supervisord_conf > /etc/supervisord.conf

 

**修改配置文件**：

在supervisord.conf最后增加：
```

[program:redis]

command = redis-server   //需要执行的命令

autostart=true    //supervisor启动的时候是否随着同时启动

autorestart=true   //当程序跑出exit的时候，这个program会自动重启

startsecs=3  //程序重启时候停留在runing状态的秒数
```

（更多配置说明请参考：[http://supervisord.org/configuration.html](http://supervisord.org/configuration.html)）

 

**运行命令**：
```

supervisord    //启动supervisor

supervisorctl   //打开命令行

[root@vm14211 ~]# supervisorctl
redis                            RUNNING    pid 24068, uptime 3:41:55

ctl中： help   //查看命令

ctl中： status  //查看状态

 ```

**遇到的问题**：

1 redis出现的不是running而是FATAL 状态

应该要去查看log

log在`/tmp/supervisord.log`

 

2 日志中显示：
```
gave up: redis entered FATAL state, too many start retries too quickly
```
修改redis.conf的daemonize为no

具体说明：[http://xingqiba.sinaapp.com/?p=240](http://xingqiba.sinaapp.com/?p=240)

事实证明webdis也有这个问题，webdis要修改的是webdis.json这个配置文件

 

完成验证：
```
1 ps aux | grep redis

[root@vm1~]# ps aux | grep redis
root     30582  0.0  0.0   9668  1584 ?        S    14:12   0:00 redis-server

 

2 kill 30582

 

3

[root@vm1 ~]# ps aux | grep redis
root     30846  0.0  0.0   9668  1552 ?        S    15:19   0:00 redis-server
```
 

看到这个时候pid更新了。

 

完成,庆祝。