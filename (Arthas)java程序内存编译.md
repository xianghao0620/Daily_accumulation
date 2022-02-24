1. 下载安装arthas
wget https://alibaba.github.io/arthas/arthas-boot.jar
或
wget https://arthas.gitee.io/arthas-boot.jar

2. 查看tomcat/java 进程ID
ps -ef |grep tomcat

3. 启动arthas
java -jar arthas-boot.jar
然后选择第二步中进程ID对应的序号


# 反编译
$ jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
# 修改文件
$ vim /tmp/UserController.java

4. sc查找加载UserController的ClassLoader
$ sc -d *UserController | grep classLoaderHash
 classLoaderHash   1be6f5c3
可以发现是spring boot的 LaunchedURLClassLoader@1be6f5c3 加载的。

5. mc内存编绎代码
保存好/tmp/UserController.java之后，使用mc(Memory Compiler)命令来编译，并且通过-c参数指定ClassLoader：
$ mc -c 1be6f5c3 /tmp/UserController.java -d /tmp
Memory compiler output:
/tmp/com/example/demo/arthas/user/UserController.class
Affect(row-cnt:1) cost in 346 ms

6. redefine热更新代码
再使用redefine命令重新加载新编译好的UserController.class：
$ redefine /tmp/com/example/demo/arthas/user/UserController.class
redefine success, size: 1
