

- [..](linux-catalog.md)

- [仓库缓存等问题](#仓库缓存等问题)
- [命令行自动补全](#命令行自动补全)
- [IP设置](#ip设置)
  - [ifcfg-eth0](#ifcfg-eth0)
- [开机自启](#开机自启)
  - [tomcat开机自启](#tomcat开机自启)
- [日志文件](#日志文件)


## 仓库缓存等问题
- Not using downloaded repomd.xml because it is older than what we have
    + yum clean all
    + 若还是有问题 可以 yum check-update
    + 如果之后执行yum有问题，删除var/cache/yum/下所有文件，再执行yum repolist all,yum makecache,yum update

## 命令行自动补全

- 安装bash-completion,退出重启登陆，或者重载配置文件即可
```
yum install epel-release -y
yum install bash-completion -y
```


## IP设置

> systemctl status network #查看网络服务状态
> /etc/sysconfig/network-scripts #网卡配置在该目录下
> ifcfg-网卡名称
> systemctl start network

### ifcfg-eth0
```
TYPE=Ethernet #网卡协议类型
DEVICE=eth0 #物理设备名称（这里只是一个逻辑名）
DEFROUTE=yes #就是default route，是否把这个eth设置为默认路由
ONBOOT=yes #系统启动时是否自动加载该网卡 yes|no
BOOTPROTO=static #获取地址协议 static|bootp|dhcp
IPADDR=192.168.10.165 # ip地址
NETMASK=255.255.0.0 # ip对应的子网掩码
GATEWAY=192.168.1.254 #ip对应的网关地址
DNS1=192.168.1.254 #指定DNS1地址
DNS2=10.165.1.6 #指定DNS2地址（有的话配，没有就不配置）
NM_CONTROLLED=no #是否由Network Manager 控制该网络接口。修改保存后立即生效，无需重启。但有坑，建议设置为no
USERCTL=yes #非ROOT用户是否允许控制整个设备 yes|no
IPV6INIT=yes #是否执行IPv6 yes:支持IPv6 no:不支持IPv6
```

## 开机自启
- centos：在/etc/rc.d/rc.local中写入命令,并赋予权限
> /usr/lib/systemd/system #service文件目录(centos)
> /etc/systemd/system #service文件目录(ubuntu)
> systemctl enable  httpd
> systemctl is-enabled  httpd
> systemctl disable  httpd
> systemctl list-unit-files|grep enabled      #查看自启动服务列表

[Service文件详解](https://blog.csdn.net/Mr_Yang__/article/details/84133783)

- .service文件内容
```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit] #启动顺序与依赖关系
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service # After：在network.target,auditd.service启动后才启动
# Before
Wants=sshd-keygen.service # Wants字段：表示sshd.service与sshd-keygen.service之间存在"弱依赖"关系
# Requires字段则表示"强依赖"关系，即如果该服务启动失败或异常退出，那么sshd.service也必须退出

[Service] #启动行为
EnvironmentFile=/etc/sysconfig/sshd #指定当前服务的环境参数文件
ExecStart=/usr/sbin/sshd -D $OPTIONS #定义启动进程时执行的命令
ExecReload=/bin/kill -HUP $MAINPID #重启服务时执行的命令
# 所有的启动设置之前，都可以加上一个连词号（-），表示"抑制错误"
Type=simple #启动类型
KillMode=process
Restart=on-failure #重启方式
RestartSec=42s

[Install] #定义如何安装这个配置文件，即怎样做到开机启动
WantedBy=multi-user.targe
```


```
[Unit]                                                                                      //对服务的说明
Description=nginx - high performance web server              //描述服务
After=network.target remote-fs.target nss-lookup.target   //描述服务类别

[Service]                                                                                 //服务的一些具体运行参数的设置
Type=forking                                                                         //后台运行的形式
PIDFile=/usr/local/nginx/logs/nginx.pid                               //PID文件的路径
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf   //启动准备
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf           //启动命令
ExecReload=/usr/local/nginx/sbin/nginx -s reload                                                 //重启命令
ExecStop=/usr/local/nginx/sbin/nginx -s stop                                                       //停止命令
ExecQuit=/usr/local/nginx/sbin/nginx -s quit                                                        //快速停止
PrivateTmp=true                                                                  //给服务分配临时空间

[Install]
WantedBy=multi-user.target                                               //服务用户的模式
```

### tomcat开机自启

1. tomcat.service文件
```service
[Unit]
Description=tomcat
After=network.target redis.service mariadb.service
Requires=mariadb.service

[Service]
Type=forking
User=cjs
Group=cjs
Environment="JAVA_HOME=/home/cjs/tools/java-se-8u40" # 如果tomcat的启动文件中加入了环境变量，此处可以删除
PIDFile=/home/cjs/tools/tomcat-9.0.30/tomcat.pid
ExecStart=/home/cjs/tools/tomcat-9.0.30/bin/startup.sh
ExecStop=/home/cjs/tools/tomcat-9.0.30/bin/shutdown.sh
ExecReload=/bin/kill -s -HUP $MAINPID
PrivateTmp=true
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
2. 在tomcat的catalina.sh开头加入环境变量与pid,如果此处不加pid,则同时删除service文件中的PIDFile
```sh
export JAVA_HOME=/home/cjs/tools/java-se-8u40
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
export CATALINA_BASE=/home/cjs/tools/tomcat-9.0.30
export CATALINA_HOME=/home/cjs/tools/tomcat-9.0.30
export CATALINA_TMPDIR=/home/cjs/tools/tomcat-9.0.30/temp

# Copy CATALINA_BASE from CATALINA_HOME if not already set  
[ -z "$CATALINA_BASE" ] && CATALINA_BASE="$CATALINA_HOME"
# 设置pid。一定要加在CATALINA_BASE定义后面，要不然pid会生成到/下面  
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
```
3. 将tomcat.service放入/lib/systemd/system/文件夹下
4. `systemctl start tomcat.service`启动；`systemctl enable tomcat.service`加入开机自启


## 日志文件
> /var/log/