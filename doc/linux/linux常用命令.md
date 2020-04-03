### Linux常用命令

```mysql
#重启mysql
service mysql restart
```

#### 防火墙

```mysql
##1:查看防火状态
systemctl status firewalld
service  iptables status
##2:暂时关闭防火墙
systemctl stop firewalld
service  iptables stop
##3:永久关闭防火墙
systemctl disable firewalld
chkconfig iptables off
##4:重启防火墙
systemctl enable firewalld
service iptables restart  
##永久关闭后重启
##暂时还没有试过
chkconfig iptables on
```

