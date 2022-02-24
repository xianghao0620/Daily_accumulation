# 一.Java相关
## 1. JDK,JRE,SDK名称解释：


```
JDK就是Java DevelopmentKit.简单的说JDK是面向开发人员使用的SDK，它提供了Java的开发环境和运行环境。
SDK就是Software Development Kit 一般指软件开发包，可以包括函数库、编译程序等。 
JRE就是Java Runtime Enviroment是指Java的运行环境，是面向Java程序的使用者，而不是开发者。
```

## 2. J2EE，J2SE，J2ME名称解释：


```
J2EE，J2SE，J2ME是Sun 公司的Java多个版本,就像Windows XP还有专业版和家庭版是一样的。

J2EE：Java 2 Platform Enterprise Edition 企业版，用于企业应用，支持分布式部署。
J2SE：Java 2 Platform Standard Edition 标准版，用于桌面应用，也是J2EE的基础。
J2ME：Java 2 Platform Micro Edition 移动版用于小型设备，是J2SE的一个子集。
```
## 3. JAVA的中间件：

```
tomcat, weblogic、webshpere，JBos

```
# 二.TOMCAT安装相关
## 1.Tomcat安装：

```
wget http://httpd-mirror.frgl.pw/apache/tomcat/tomcat-8/v8.0.44/bin/apache-tomcat-8.0.44.tar.gz
下载 jdk-8u131-linux-x64.tar.gz

useradd -u 601 tomca    #创建tomcat用户

tar -zxvf jdk-8u131-linux-x64.tar.gz
mv jdk1.8.0_131 /usr/local/
ln -s /usr/local/jdk1.8.0_131/ /usr/local/jdk

tar -zxvf apache-tomcat-8.0.44.tar.gz
mv apache-tomcat-8.0.44 /usr/local/
ln -s  /usr/local/apache-tomcat-8.0.44 /usr/local/tomcat
chown -R tomcat:tomcat /usr/local/tomcat/
```
## 2.配置JAVA的环境变量：

vim /etc/profile #底部追加

```
export JAVA_HOME=/usr/local/jdk1.8.0_131/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export TOMCAT_HOME=/usr/local/tomcat
```
source /etc/profile

## 3.启动Tomcat：

```
su - tomcat
/usr/local/tomcat/bin/startup.sh

http://172.16.1.211:8080/ #访问地址
```
## 4.配置文件详解：

```
<?xml version='1.0' encoding='utf-8'?>
<!--
<Server>元素代表整个容器,是Tomcat实例的顶层元素.由org.apache.catalina.Server接口来定义.它包含一个<Service>元素.并且它不能做为任何元素的子元素.
    port指定Tomcat监听shutdown命令端口.终止服务器运行时,必须在Tomcat服务器所在的机器上发出shutdown命令.该属性是必须的.
    shutdown指定终止Tomcat服务器运行时,发给Tomcat服务器的shutdown监听端口的字符串.该属性必须设置
-->
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <!--service服务组件-->
  <Service name="Catalina">
    <!--
    connector：接收用户请求，类似于httpd的listen配置监听端口.
        port指定服务器端要创建的端口号，并在这个端口监听来自客户端的请求。
        address：指定连接器监听的地址，默认为所有地址（即0.0.0.0）
        protocol连接器使用的协议，支持HTTP和AJP。AJP（Apache Jserv Protocol）专用于tomcat与apache建立通信的， 在httpd反向代理用户请求至tomcat时使用（可见Nginx反向代理时不可用AJP协议）。
        minProcessors服务器启动时创建的处理请求的线程数
        maxProcessors最大可以创建的处理请求的线程数
        enableLookups如果为true，则可以通过调用request.getRemoteHost()进行DNS查询来得到远程客户端的实际主机名，若为false则不进行DNS查询，而是返回其ip地址
        redirectPort指定服务器正在处理http请求时收到了一个SSL传输请求后重定向的端口号
        acceptCount指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理
        connectionTimeout指定超时的时间数(以毫秒为单位)
    -->
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <!--engine,核心容器组件,catalina引擎,负责通过connector接收用户请求,并处理请求,将请求转至对应的虚拟主机host
        defaultHost指定缺省的处理请求的主机名，它至少与其中的一个host元素的name属性值是一样的
    -->
    <Engine name="Catalina" defaultHost="localhost">
      <!--Realm表示存放用户名，密码及role的数据库-->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <!--
      host表示一个虚拟主机
        name指定主机名
        appBase应用程序基本目录，即存放应用程序的目录.一般为appBase="webapps" ，相对于CATALINA_HOME而言的，也可以写绝对路径。
        unpackWARs如果为true，则tomcat会自动将WAR文件解压，否则不解压，直接从WAR文件中运行应用程序
        autoDeploy：在tomcat启动时，是否自动部署。
        xmlValidation：是否启动xml的校验功能，一般xmlValidation="false"。
        xmlNamespaceAware：检测名称空间，一般xmlNamespaceAware="false"。
      -->
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <!--
        Context表示一个web应用程序，通常为WAR文件
            docBase应用程序的路径或者是WAR文件存放的路径,也可以使用相对路径，起始路径为此Context所属Host中appBase定义的路径。
            path表示此web应用程序的url的前缀，这样请求的url为http://localhost:8080/path/****
            reloadable这个属性非常重要，如果为true，则tomcat会自动检测应用程序的/WEB-INF/lib 和/WEB-INF/classes目录的变化，自动装载新的应用程序，可以在不重启tomcat的情况下改变应用程序
        -->
        <Context path="" docBase="" debug=""/>
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```
## 5.配置tomcat的管理界面用户（生产一般删掉留个监控页面）：
vim conf/tomcat-users.xml

```
#配置两个角色
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
#配置tomcat用户
  <user username="tomcat" password="wmj123" roles="manager-gui,admin-gui"/>
```
usr/local/tomcat/bin/shutdown.sh
/usr/local/tomcat/bin/startup.sh

```
访问地址： http://172.16.1.211:8080/host-manager/html 账号：tomcat 密码：wmj123
```
# 三.生产环境Tomcat调优

## 1 .屏蔽DNS查询:
Web应用程序可以通过Web容器提供的getRemoteHost()方法获得访问Web应用客户的IP地址和名称，但是这样会消耗Web容器的资源，并且还需要通过IP地址和DNS服务器反查用户的名字。因此当系统上线时，可以将这个属性关闭，从而减少资源消耗，那么Web应用也就只能记录下IP地址。修改的属性是enableLoopups=”false”。

## 2 调整线程数:
Tomcat通过线程池来为用户访问提供响应，对于上线的系统初步估计用户并发数量后，再调整线程池容量。例如，用户并发数量在100左右时，可以设置minProcessors=”100”，maxProcessors=”100”。将最大和最小设置为一样后，线程池不会再释放空闲的线程，当用户访问突然增加时，不需要再消耗系统资源去创建新的线程。

## 3 调整最大连接数:
这个其实最复杂，即使用户并发量大，但是系统反应速度快，也没必要把这个值设置太高，高了系统需要消耗大量的资源去切换线程，但是如果设置太低也会造成应用无法满足用户并发需要。因此设置这个最好能够结合整个系统的跟踪与调优，使系统达到最好的平稳状态，一般设置为maxProcessors的1.5倍即可。

## 4 压缩管理:

```
tomcat作为一个应用服务器，也是支持 gzip 压缩功能的。我们可以在 server.xml 配置文件中的 Connector 节点中配置如下参数，来实现对指定资源类型进行压缩。

```
## 5 生产配置实例:
vim conf/server.xml

```
<Connector port="8080" protocol="HTTP/1.1"
        URIEncoding="UTF-8"    #设置编码
        minSpareThreads="25"  #Tomcat初始化时创建的 socket线程数
        maxSpareThreads="75"  #Tomcat连接器的最大空闲socket 线程数，一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。默认值50
        enableLookups="false"  #屏蔽DNS查询
        disableUploadTimeout="true"  #该标志位表明当执行servlet时，是否允许servlet容器使用一个不同的、更长的连接超时。启用该标志位将导致在上传数据时，要么使用更长的时间完成上传，要么出现更长的超时。如果不指定，该属性为“false”。       
        connectionTimeout="20000"   #网络超时时间
        acceptCount="300"     #当线程数达到maxThreads后，后续请求会被放入一个等待队列，这个acceptCount是这个队列的大小，满了之后客户请求会被拒绝.
        maxThreads="300"     #这个值表示Tomcat可创建的最大的线程数，即最大并发数，默认值为“200”
        maxProcessors="1000"   #最大连接线程数，即：并发处理的最大请求数，默认值为75 ，一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程
        minProcessors="5"      #最小空闲连接线程数，用于提高系统处理性能，默认值为10
        useURIValidationHack="false"  #可以减少它对一些url的不必要的检查从而减省开销。
        <!--   前端使用nginx作为反向代理，不需要启用tomcat压缩功能。
        compression="on"   #打开压缩功能
        compressionMinSize="2048" #启用压缩的输出内容大小，这里面默认为2KB
        compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"   #压缩类型
        -->
        redirectPort="8443"/>
```
## 6.生产JVM参数调优：
优化catalina.sh配置文件。在catalina.sh配置文件中添加以下代码：

```
JAVA_OPTS="-server -Dfile.encoding=UTF-8 -Xms1024m -Xmx1024m -XX:PermSize=128m -XX:MaxPermSize=128m -Djava.awt.headless=true"

-server:一定要作为第一个参数，在多个CPU时性能佳
-Xms：初始堆内存Heap大小，使用的最小内存,cpu性能高时此值应设的大一些
-Xmx：初始堆内存heap最大值，使用的最大内存
上面两个值是分配JVM的最小和最大内存，取决于硬件物理内存的大小，建议均设为物理内存的一半或者1/4。
-XX:PermSize:设定内存的永久保存区域,建议设置128m,如果java包很大，可以相应扩大。
-XX:MaxPermSize:设定最大内存的永久保存区域,建议设置128m，如果java包很大，可以相应扩大。
------- 下面的可以使用默认或者根据应用调整 -----------
-Xss 15120 这使得JBoss每增加一个线程（thread)就会立即消耗15M内存，而最佳值应该是128K,默认值好像是512k.
+XX:AggressiveHeap 会使得 Xms没有意义。这个参数让jvm忽略Xmx参数,疯狂地吃完一个G物理内存,再吃尽一个G的swap。
-Xss：每个线程的Stack大小
-verbose:gc 现实垃圾收集信息
-Xloggc:gc.log 指定垃圾收集日志文件
-Xmn：young generation的heap大小，一般设置为Xmx的3、4分之一
-XX:+UseParNewGC ：缩短minor收集的时间
-XX:+UseConcMarkSweepGC ：缩短major收集的时间
```
# 四.生产环境Tomcat安全规范
## 1.更改服务监听端口
若 Tomcat 都是放在内网的，则针对 Tomcat 服务的监听地址都是内网地址

```
标准配置：<Connector port="10000" server="webserver"/>
```
## 2.telnet管理端口保护

```
修改默认的 8005 管理端口不易猜测（大于1024），但要求端口配置在8000~8999之间

修改SHUTDOWN命令为其他字符串 
标准配置：<Server port="8578" shutdown="dangerous">

```
## 3.AJP连接端口的保护

```
修改默认的ajp 8009端口为不易冲突（大于1024），但要求端口配置在8000~8999之间

通过iptables规则限制ajp端口访问的权限仅为线上机器，目的在于防止线下测试流量被apache的mod_jk转发至线上tomcat服务器

标准配置：<Connector port="8349" protocol="AJP/1.3"/>

```
## 4.禁用管理端

```
删除默认$CATALINA_HOME/conf/tomcat-users.xml文件，重启tomcat将会自动生成新的文件

删除$CATALINA_HOME/webapps下载默认的所有目录和文件

将tomcat应用根目录配置为tomcat安装目录以外的目录

```
## 5.隐藏Tomcat的版本信息(可以不做)
针对该信息的显示是由一个jar包控制的，该jar包存放在$CATALINA_HOME/lib目录下，名称为 catalina.jar，通过 jar xf 命令解压这个 jar 包会得到两个目录 META-INF 和 org ,修改 org/apache/catalina/util/ServerInfo.properties 文件中的 serverinfo 字段来实现来更改我们tomcat的版本信息


```
$ cd $CATALINA_HOME/lib
$ jar xf catalina.jar
$ cat org/apache/catalina/util/ServerInfo.properties |grep -v '^$|#'
$ mkdir -p org/apache/catalina/util
$ vim ServerInfo.properties
server.info=nolinux        # 把这个值改成其它值就行了

自定义错误页面：修改$CATALINA_HOME/conf/web.xml重定向 403/404/500等错误到指定的错误页面
```
## 6.降权启动
Tomcat启动用户权限必须非root权限，尽量降低tomcat启动用户的目录访问权限，如需直接对外使用80端口，可通过普通账号启动后，配置iptables规则进行转发，为了防止 Tomcat 被植入 web shell 程序后，可以修改项目文件。要将 Tomcat 和项目的属主做分离，即便被破坏也无法创建和编辑项目文件。

## 7.文件列表访问控制
```
$CATALINA_HOME/conf/web.xml文件中的default部分的listings的配置必须为false(默认)，表示不列出文件列表

```
## 8.访问限制（可以用nginx做）
通过配置，限定访问的IP来源

全局设置限定IP和域名访问：

```
<Host name="localhost"  appBase="/data/www/tomcat_webapps"   unpackWARs="true" autoDeploy="false">
   <Valve className="org.apache.catalina.valves.RemoteAddrValve"  allow="192.168.1.10,192.168.1.30,192.168.2.*" deny=""/>  
   <Valve className="org.apache.catalina.valves.RemoteHostValve"  allow="www.test.com,*.test.com" deny=""/>
</Host>
```
## 9.脚本权限回收

```
控制CATALINAHOME/bin目录下的start.sh、catalina.sh、shutdown.sh的可执行权限，chmod−R744CATALINA_HOME/bin/*

```
## 10.访问日志格式规范
开启tomcat默认访问日志中Referer和User-Agent记录

```
标准配置：
<Valve className="org.apache.catalina.valves.AccessLogValve"
   directory="logs" prefix="localhost_access_log"
     suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b %{Referer}i %{User-Agent}i %D"
     resolveHosts="false" />
```

## 11.Server header重写

```
在HTTP Connector配置中加入server的配置，server=”JDX-server”

```
# 五. Tomcat的监控
# 1.Zabbix监控JVM和Tomcat性能：

```
zabbix可以使用zabbix-java-gateway监控jvm/tomcat性能
```
[Zabbix监控JVM和Tomcat性能](http://www.mamicode.com/info-detail-1521653.html)
