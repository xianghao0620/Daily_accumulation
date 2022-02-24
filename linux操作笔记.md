linux日志清理步骤
cd   /     切换到根路径
du -sh * | sort -nr         按照文件大小排序  ，一级级向下看 ，如果是日志文件删掉就行，
清空日志文件：> catalina.out

linux-压缩
tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg
tar –czf jpg.tar.gz *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar –cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux
zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux

linux-解压
tar –xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar –xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip

打包成tar.gz格式压缩包
tar -zcvf renwolesshel.tar.gz /renwolesshel

解压tar.gz格式压缩包
tar zxvf renwolesshel.tar.gz
一.本地服务器copy到远程服务器的方法
 
scp /root/cbs8/temp/cbs.txt root@IP：/root/cbs8
 
1./root/cbs8/temp/cbs.txt  本地路径要copy到远程服务器的文件

2.root@IP：/root/cbs8 远程服务器路径 （注明：root是远程服务器用户名 IP是远程服务器IP地址）
 
二.远程服务器copy到本地服务器的方法
scp  root@IP:/root/cbs8/temp/cbs.txt /root/cbs8


日志文件内容截图
sed -n '/2020-11-26 10:00/, /2020-11-26 12:00/p' catalina.out > log.txt

sh文件赋予执行权限
chmod u+x file.sh

linux关闭防火墙命令

一、下面是red hat/CentOs7关闭防火墙的命令!
1:查看防火状态
systemctl status firewalld
service  iptables status
2:暂时关闭防火墙
systemctl stop firewalld
service  iptables stop
3:永久关闭防火墙
systemctl disable firewalld
chkconfig iptables off
4:重启防火墙
systemctl enable firewalld
service iptables restart  
5:永久关闭后重启
//暂时还没有试过
chkconfig iptables on
二、firewalld
Centos7默认安装了firewalld，如果没有安装的话，可以使用 yum install firewalld firewalld-config进行安装。
1.启动防火墙
systemctl start firewalld 
2.禁用防火墙
systemctl stop firewalld
3.设置开机启动
systemctl enable firewalld
4.停止并禁用开机启动
sytemctl disable firewalld
5.重启防火墙
 
firewall-cmd --reload

6.查看状态
systemctl status firewalld或者 firewall-cmd --state
7.查看版本
firewall-cmd --version
8.查看帮助
firewall-cmd --help
9.查看区域信息
firewall-cmd --get-active-zones
10.查看指定接口所属区域信息
firewall-cmd --get-zone-of-interface=eth0
11.拒绝所有包
firewall-cmd --panic-on
12.取消拒绝状态
firewall-cmd --panic-off
13.查看是否拒绝
firewall-cmd --query-panic
14.将接口添加到区域(默认接口都在public)
firewall-cmd --zone=public --add-interface=eth0(永久生效再加上 --permanent 然后reload防火墙)
15.设置默认接口区域
firewall-cmd --set-default-zone=public(立即生效，无需重启)
16.更新防火墙规则
firewall-cmd --reload或firewall-cmd --complete-reload(两者的区别就是第一个无需断开连接，就是firewalld特性之一动态
添加规则，第二个需要断开连接，类似重启服务)
17.查看指定区域所有打开的端口
 
firewall-cmd --zone=public --list-ports
18.在指定区域打开端口（记得重启防火墙）
 
firewall-cmd --zone=public --add-port=80/tcp(永久生效再加上 --permanent)

说明：
–zone 作用域
–add-port=8080/tcp 添加端口，格式为：端口/通讯协议
–permanent #永久生效，没有此参数重启后失效
补充：
CentOS 7 下使用 Firewall 
在 CentOS 7 中，引入了一个新的服务，Firewalld，下面一张图，让大家明确的了解 Firewall 与 iptables 之间的关系与区别。

安装它，只需
# yum install firewalld

如果需要图形界面的话，则再安装
# yum install firewall-config

一、介绍
防火墙守护 firewalld 服务引入了一个信任级别的概念来管理与之相关联的连接与接口。它支持 ipv4 与 ipv6，并支持网桥，采用 firewall-cmd (command) 或 firewall-config (gui) 来动态的管理 kernel netfilter 的临时或永久的接口规则，并实时生效而无需重启服务。
zone
Firewall 能将不同的网络连接归类到不同的信任级别，Zone 提供了以下几个级别
drop: 丢弃所有进入的包，而不给出任何响应
block: 拒绝所有外部发起的连接，允许内部发起的连接
public: 允许指定的进入连接
external: 同上，对伪装的进入连接，一般用于路由转发
dmz: 允许受限制的进入连接
work: 允许受信任的计算机被限制的进入连接，类似 workgroup
home: 同上，类似 homegroup
internal: 同上，范围针对所有互联网用户
trusted: 信任所有连接
过滤规则
source: 根据源地址过滤
interface: 根据网卡过滤
service: 根据服务名过滤
port: 根据端口过滤
icmp-block: icmp 报文过滤，按照 icmp 类型配置
masquerade: ip 地址伪装
forward-port: 端口转发
rule: 自定义规则
其中，过滤规则的优先级遵循如下顺序
source
interface
firewalld.conf
二、使用方法
# systemctl start firewalld         # 启动,
# systemctl enable firewalld        # 开机启动
# systemctl stop firewalld          # 关闭
# systemctl disable firewalld       # 取消开机启动
具体的规则管理，可以使用 firewall-cmd，具体的使用方法可以
$ firewall-cmd --help
 
--zone=NAME                         # 指定 zone
--permanent                         # 永久修改，--reload 后生效
--timeout=seconds                   # 持续效果，到期后自动移除，用于调试，不能与 --permanent 同时使用
1. 查看规则
查看运行状态
$ firewall-cmd --state

查看已被激活的 Zone 信息
$ firewall-cmd --get-active-zones
public
  interfaces: eth0 eth1
查看指定接口的 Zone 信息
$ firewall-cmd --get-zone-of-interface=eth0
public
查看指定级别的接口
$ firewall-cmd --zone=public --list-interfaces
eth0
查看指定级别的所有信息，譬如 public
$ firewall-cmd --zone=public --list-all
public (default, active)
  interfaces: eth0
  sources:
  services: dhcpv6-client http ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
查看所有级别被允许的信息
$ firewall-cmd --get-service

查看重启后所有 Zones 级别中被允许的服务，即永久放行的服务
$ firewall-cmd --get-service --permanent

2. 管理规则
# firewall-cmd --panic-on           # 丢弃
# firewall-cmd --panic-off          # 取消丢弃
# firewall-cmd --query-panic        # 查看丢弃状态
# firewall-cmd --reload             # 更新规则，不重启服务
# firewall-cmd --complete-reload    # 更新规则，重启服务
添加某接口至某信任等级，譬如添加 eth0 至 public，永久修改
# firewall-cmd --zone=public --add-interface=eth0 --permanent

设置 public 为默认的信任级别
# firewall-cmd --set-default-zone=public

a. 管理端口
列出 dmz 级别的被允许的进入端口
# firewall-cmd --zone=dmz --list-ports

允许 tcp 端口 8080 至 dmz 级别
# firewall-cmd --zone=dmz --add-port=8080/tcp

允许某范围的 udp 端口至 public 级别，并永久生效
# firewall-cmd --zone=public --add-port=5060-5059/udp --permanent

b. 网卡接口
列出 public zone 所有网卡
# firewall-cmd --zone=public --list-interfaces

将 eth0 添加至 public zone，永久
# firewall-cmd --zone=public --permanent --add-interface=eth0

eth0 存在与 public zone，将该网卡添加至 work zone，并将之从 public zone 中删除
# firewall-cmd --zone=work --permanent --change-interface=eth0

删除 public zone 中的 eth0，永久
# firewall-cmd --zone=public --permanent --remove-interface=eth0

c. 管理服务
添加 smtp 服务至 work zone
# firewall-cmd --zone=work --add-service=smtp

移除 work zone 中的 smtp 服务
# firewall-cmd --zone=work --remove-service=smtp

d. 配置 external zone 中的 ip 地址伪装
查看
# firewall-cmd --zone=external --query-masquerade

打开伪装
# firewall-cmd --zone=external --add-masquerade

关闭伪装
# firewall-cmd --zone=external --remove-masquerade

e. 配置 public zone 的端口转发
要打开端口转发，则需要先
# firewall-cmd --zone=public --add-masquerade

然后转发 tcp 22 端口至 3753
# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=3753

转发 22 端口数据至另一个 ip 的相同端口上
# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toaddr=192.168.1.100

转发 22 端口数据至另一 ip 的 2055 端口上
# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=2055:toaddr=192.168.1.100

f. 配置 public zone 的 icmp
查看所有支持的 icmp 类型
# firewall-cmd --get-icmptypes
destination-unreachable echo-reply echo-request parameter-problem redirect router-advertisement router-solicitation source-quench time-exceeded
列出
# firewall-cmd --zone=public --list-icmp-blocks

添加 echo-request 屏蔽
# firewall-cmd --zone=public --add-icmp-block=echo-request [--timeout=seconds]

移除 echo-reply 屏蔽
# firewall-cmd --zone=public --remove-icmp-block=echo-reply

g. IP 封禁
# firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='222.222.222.222' reject"

当然，我们仍然可以通过 ipset 来封禁 ip
封禁 ip
# firewall-cmd --permanent --zone=public --new-ipset=blacklist --type=hash:ip
# firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=222.222.222.222
封禁网段
# firewall-cmd --permanent --zone=public --new-ipset=blacklist --type=hash:net
# firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=222.222.222.0/24
导入 ipset 的 blacklist 规则
# firewall-cmd --permanent --zone=public --new-ipset-from-file=/path/blacklist.xml

如果已经存 blacklist，则需要先删除
# firewall-cmd --get-ipsets
blacklist
# firewall-cmd --permanent --zone=public --delete-ipset=blacklist
然后封禁 blacklist
# firewall-cmd --permanent --zone=public --add-rich-rule='rule source ipset=blacklist drop'

重新载入以生效
# firewall-cmd --reload

查看 blacklist
# firewall-cmd --ipset=blacklist --get-entries




linux 清空catalina.out日志 不需要重启tomcat（五种方法）
linux 清空catalina.out日志 不需要重启tomcat
 1.重定向方法清空文件
 
[root@localhost logs]# du -h catalina.out  查看文件大小
17M catalina.out
[root@localhost logs]# > catalina.out   重定向清空文件
[root@localhost logs]# du -h catalina.out  查看文件大小
0 catalina.out
2.使用true命令重定向清空文件
 
[root@localhost logs]# du -h catalina.out
448K catalina.out
[root@localhost logs]# true > catalina.out
[root@localhost logs]# du -h catalina.out
0 catalina.out
 
3、使用cat/cp/dd命令及/dev/null设备来清空文件
 cat  /dev/null 命令清空文件

 
 cp  /dev/null

 
dd if=/dev/null of=catalina.out

4、使用echo命令清空文件
echo -n  " " > catalina.out   ==》加上"-n"参数，默认情况下会"\n"，也就是回车符

 
5、使用truncate命令清空文件
truncate -s 0 catalina.out   -s参数是设置文件的大小，清空文件的话，就设定为0



linux下开启防火墙，允许通过的端口
--------------------------------------------------------------------------------
1、查看防火墙状态
systemctl status firewalld
 
2、如果不是显示active状态，需要打开防火墙
systemctl start firewalld
3、# 查看所有已开放的临时端口（默认为空）
# firewall-cmd --list-ports
# 查看所有永久开放的端口（默认为空）
# firewall-cmd --list-ports --permanent

# 添加临时开放端口（例如：比如我修改ssh远程连接端口是223，则需要开放这个端口）
# firewall-cmd --add-port=223/tcp
# 添加永久开放的端口（例如：223端口）
# firewall-cmd --add-port=223/tcp --permanent
# 关闭临时端口
# firewall-cmd --remove-port=80/tcp
# 关闭永久端口
# firewll-cmd --remove-port=80/tcp --permanent
# 配置结束后需要输入重载命令并重启防火墙以生效配置
# firewall-cmd --reload
# systemctl restart firewalld
