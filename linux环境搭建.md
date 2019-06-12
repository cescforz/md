linux环境搭建

```shell
#centos常用命令安装
#上传(rz)下载(sz)
yum install lrzsz

yum install -y git
```



```shell
#常用命令
#1.退出文本编辑命令-->ESC,跳到命令模式：
:w   保存文件但不退出vi
:w file 将修改另外保存到file中，不退出vi
:w!   强制保存，不推出vi
:wq  保存文件并退出vi
:wq! 强制保存文件，并退出vi
:q 不保存文件，退出vi
:q! 不保存文件，强制退出vi
:e! 放弃所有修改，从上次保存文件开始再编辑
```



```shell
#安装java
#1.解压
tar -zxvf jdk-11_linux-x64_bin.tar.gz
#2.配置环境变量
vim /etc/profile
#3.在文件的末尾处添加以下内容
export JAVA_HOME=/apps/svr/jdk-11
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
#4.使配置文件生效
source /etc/profile
#5.检查是否生效
java -version
javac -version

#安装maven
#1.解压
tar -zxvf apache-maven-3.6.0-bin.tar.gz
#2.配置环境变量
vim /etc/profile
export M2_HOME=/apps/svr/apache-maven-3.6.0
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
#3.使配置文件生效
source /etc/profile
#4.检查是否生效
mvn -version
#5.修改配置文件,maven仓库路径

#防火墙操作：firewall 防火墙
#1、查看 firewall 服务状态
#出现 Active: active (running) 切高亮显示则表示是启动状态
#出现 Active: inactive (dead) 灰色表示停止
systemctl status firewalld
#2、查看 firewall 的状态
firewall-cmd --state
# 开启
service firewalld start(/bin/systemctl start firewalld.service)
# 重启
service firewalld restart
# 关闭
service firewalld stop(/bin/systemctl stop firewalld.service)
#查看防火墙规则
firewall-cmd --list-all 
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放 80 端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙 (修改配置后要重启防火墙)
firewall-cmd --reload

firewalld 服务被锁定，不能添加对应端口
# 执行命令，即可实现取消服务的锁定
systemctl unmask firewalld
#下次需要锁定该服务时执行
systemctl mask firewalld

# 参数解释
#1、firwall-cmd：是 Linux 提供的操作 firewall 的一个工具；
#2、--permanent：表示设置为持久；
#3、--add-port：标识添加的端口；


#安装docker
#1.更新YUM源
yum update

```

firewall-cmd --permanent --add-port=5000/tcp

firewall-cmd --permanent --add-port=9600/tcp

firewall-cmd --reload

firewall-cmd --list-all

[http://47.107.252.230](http://47.107.252.230/)

/apps/svr/elk/logstash/conf.d