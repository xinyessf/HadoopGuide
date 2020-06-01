### 学习前提

### 从其他直接copy集群

```
删除vm路径的日志文件，然后用vm虚拟机打开这个.vmx
```

**如何启用多个vm**

```
//1.设置不同ip
//2.修改配置
```

### vm如何配置

```shell
//1.修改
vi /etc/udev/rules.d/70-persistent-ipoib.rules 
//2.修改ip
vi /etc/sysconfig/network-scripts/ifcfg-eth0
MAC地址:
00:50:56:33:5E:B5
00:50:56:27:70:52
00:50:56:2a:af:c2
00:50:56:36:39:FA
--------
spark
00:50:56:2E:24:38
----
ip
getway
//3.修改主机名
vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=hdp-03
##关闭防火墙
systemctl stop firewalld.service
##
C:\Windows\System32\drivers\etc\hosts
ipconfig /flushdns
```

### ssh无密码连接

[免密登录设置](https://blog.csdn.net/qq_42493452/article/details/86214090)

```shell
##
/root/.ssh
##创建
ssh localhost
#
ssh-keygen -t rsa  
at id_rsa.pub >> authorized_keys    # 加入授权
chmod 600 authorized_keys    # 修改文件权限，如果不修改文件权限，那么其它用户就能查看该授权
##复制 ,注意没有目录要先创建
scp /root/.ssh/authorized_keys root@hdp-02:~/.ssh
scp /root/.ssh/authorized_keys root@hdp-03:~/.ssh
scp /root/.ssh/authorized_keys root@hdp-04:~/.ssh
##
scp /root/.ssh/authorized_keys root@spark-02:~/.ssh

scp /root/.ssh/authorized_keys root@node-136:~/.ssh
```

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0
00:50:56:3A:E7:8F
00:50:56:3A:E7:8F
###
#DEVICE="eth0"
BOOTPROTO="static"
HWADDR="00:0C:29:ED:85:FE"
IPV6INIT="yes"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID="2a76c2f8-cd47-44af-936d-11559b3a498d"
IPADDR="192.168.73.128"
NETMASK="255.255.255.0"

GATEWAY="192.168.73.1"
DNS1=114.114.114.114
DNS2=8.8.8.8

```

