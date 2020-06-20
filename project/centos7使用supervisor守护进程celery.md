`supervisor`是一个用`python`语言编写的进程管理工具，它可以很方便的监听、启动、停止、重启一个或多个进程。当一个进程意外被杀死，`supervisor`监听到进程死后，可以很方便的让进程自动恢复，不再需要程序员或系统管理员自己编写代码来控制。

相关名词：

```bash
supervisor     # 要安装的软件的名称。
supervisord    # 装好supervisor软件后，supervisord用于启动supervisor服务。
supervisorctl   # 用于管理supervisor配置文件中program。
```



### 下载安装

安装有两种方式：

- 使用`yum`命令安装（推荐）。
- 使用`pip`安装（不推荐）。

这里我们使用`yum`命令安装。

```bash
# 切换为root用户
su
# 下载supervisor
yum install -y supervisor
# 开机自启动
systemctl enable supervisord
# 启动supervisord服务
systemctl start supervisord
# 查看supervisord服务状态
systemctl status supervisord 
# 查看是否存在supervisord进程
ps -ef|grep supervisord 
```
![](http://cdn.qmpython.com/centos7使用supervisor守护进程celery_1585833150000.png)


### 配置文件

`supervisord` 的配置 文件是 `/etc/supervisord.conf`。
自定义配置文件目录是`/etc/supervisord.d`,该目录下文件已`.ini`为后缀。

`supervisord.conf`配置文件说明：

```bash
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

;[inet_http_server]         ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user              ;登录管理后台的用户名
;password=123               ;登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord

; [program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程

;包含其它配置文件
[include]
files = /etc/supervisord.d/*.ini ;可以指定一个或多个以.ini结束的配置文件
```

我们在`supervisord.d`下新建配置文件`celery_worker.ini`，内容：

```bash
[program:celery_worker] #celery_worker  为程序的名称
# 项目所在的根目录，这个是选择在那个目录下执行命令
directory=/root/src/www/QmpythonBlog
#  要执行的操作，一旦进程没了，通过这个操作重新启动进程。这句就和之前启动celery work一样。注意虚拟环境
command=/root/.virtualenvs/qmpython_env/bin/celery -A celery_tasks.main worker -l info

# 用户
user=root

stopsignal=INT
# 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
startsecs=10
# 启动失败自动重试次数，默认是 3
startretries=3
stopwaitsecs=0

# 是否自启动，在supervisord启动的时候也自动启动
autostart=true
# 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
autorestart=true
# 自动重启时间间隔（s）
startsecs=3
# 错误日志文件
stderr_logfile= /root/src/logback/supervisord/celery_worker.err
# 输出日志文件
stdout_logfile= /root/src/logback/supervisord/celery_worker.log
```

执行命令使用心得配置文件运行supervisor服务：

```bash
supervisorctl reload  # 重新加载配置文件
```

![](http://cdn.qmpython.com/centos7使用supervisor守护进程celery_1585833234000.png)

至此，我们就实现了以后台方式运行celery worker。



### 其他配置

如果要启动定时任务监视器，则配置如下：
`celery beat --loglevel=info # 启动定时任务监听器`；

```bash
[root@gitlab conf.d]# cat celery_beat.ini
[program:CeleryBeat]  
#CelertBeat 为程序的名称
command=/root/.envs/hrm/bin/python manage.py celery beat --loglevel=info
#需要执行的命令
directory=/root/TestProject/HttpRunnerManager
#命令执行的目录
#environment=ASPNETCORE__ENVIRONMENT=Production
#环境变量
user=root 
#用户
stopsignal=INT
autostart=true
#是否自启动
autorestart=true
#是否自动重启
startsecs=3
#自动重启时间间隔（s）
stderr_logfile=/root/TestProject/logs/celerybeat.err.log
#错误日志文件
stdout_logfile=/root/TestProject/logs/celerybeat.out.log
#输出日志文件
```



`celery flower --address=0.0.0.0 --port=5555 # 启动任务监控后台`；

```bash
[root@gitlab conf.d]# cat celery_flower.ini
[program:CeleryFlower]  
#CeleryFlower  为程序的名称
command=/root/.envs/hrm/bin/celery flower --address=0.0.0.0 --port=5555
#需要执行的命令
directory=/root/TestProject
#命令执行的目录
#environment=ASPNETCORE__ENVIRONMENT=Production
#环境变量
user=root 
#用户
stopsignal=INT
autostart=true
#是否自启动
autorestart=true
#是否自动重启
startsecs=3
#自动重启时间间隔（s）
stderr_logfile=/root/TestProject/logs/celeryflower.err.log
#错误日志文件
stdout_logfile=/root/TestProject/logs/celeryflower.out.log
#输出日志文件
```

