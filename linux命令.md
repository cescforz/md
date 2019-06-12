### linux命令

```sh
日志：
Linux 下查看日志时，使用tail -f可以不断的刷新日志信息。
例如:tail -f logs.log 
暂停刷新:ctrl+s
继续终端:ctrl+q
退出:ctrl+c

tail Console.log
    输出文件最后10行的内容
tail -nf Console.log  --n为最后n行
    输出文件最后n行的内容，同时监视文件的改变，只要文件有一变化就同步刷新并显示出来
tail -n 5 filename
    输出文件最后5行的内容
tail -f filename
输出最后10行内容，同时监视文件的改变，只要文件有一变化就显示出来


cat -n filename |grep "关键字"
sed -n '/起始时间/,/结束时间/p' filename

=================================================
上传下载：
下载：sz

=================================================
查看系统中文件的使用情况
df -h
查看当前目录下各个文件及目录占用空间大小
du -sh *

方法一：切换到要删除的目录，删除目录下的所有文件
rm -f *

方法二：删除logs文件夹下的所有文件，而不删除文件夹本身
rm -rf log/*


查看端口是否可访问：telnet ip 端口号 （如本机的35465：telnet localhost 35465）


===================================================
防火墙
vi /etc/sysconfig/iptables
systemctl restart iptables.service. 重启防火墙使配置生效
systemctl enable iptables.service  设置防火墙开机启动
```



多版本java 安装

```
解压包
tar -zxvf jdk-7u51-linux-x64.tar.gz

用管理员用户root对 
vim /etc/profile

export JAVA_HOME=/apps/svr/jdk1.8
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

安装不同版本Java
alternatives --install /usr/bin/java java /apps/svr/jdk1.8/bin/java 1
alternatives --install /usr/bin/java java /apps/svr/jdk11/bin/java 2

使环境变量设置立即生效
source /etc/profile

选择jdk对应的数字--切换jdk版本
alternatives --config java
```

docker

```shell
docker inspect 容器ID | grep IPAddress

#docker 列出每个容器的IP
docker inspect -f='{{.Name}} {{.NetworkSettings.IPAddress}} {{.HostConfig.PortBindings}}' $(docker ps -aq)
```

