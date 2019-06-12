###docker mongodb

```shell
mongodb操作
1.搜索 MongoDB 镜像
docker search mongodb
2.拉取 MongoDB 镜像
docker pull mongo
3.查看本地 docker 镜像 
docker images
4.启动容器
#简化版
docker run --name mongo1 -p 21117:27017 -d mongo --noprealloc --smallfiles

#自定义mongo数据路径
docker run --name mongo_rs1 -v ~/test/mongo_sr1:/mongodb -p 37117:27017 -d mongo mongod --logpath /mongodb/mongo.log  --logappend --dbpath /mongodb

#开启mongo双主配置
docker run --name mongo_rs2 -v ~/test/mongo_sr2:/mongodb -p 37118:27017 -d mongo mongod --logpath /mongodb/mongo.log  --logappend --dbpath /mongodb --master --slave --port=27017 --autoresync  --source=192.168.118.73:37117

#常规，用mongo默认数据路径，并挂载出来
docker run  --name mongodb_alpha -d --dns=192.168.1.26 -p 37017:27017 -v /home/devsa_dev/mongo_data/configdb:/data/configdb -v /home/devsa_dev/mongo_data/db:/data/db mongo

==================================================================================================
开启 mongo 容器密码认证
docker run --name mongo -p 27017:27017 -d mongo --noprealloc --smallfiles --auth

5.进入容器
docker exec -it mongo bash
6.进入bin目录
cd bin
7.启动
mongo
8.进入admin数据库
use admin 
9.创建管理员账户
db.createUser({ user: "admin", pwd: "admin", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
#mongodb中的用户是基于身份role的，该管理员账户的 role是 userAdminAnyDatabase。 'userAdmin'代表用户管理身份，'AnyDatabase'代表可以管理任何数据库
10.验证第3步用户添加是否成功
db.auth("admin", "admin") 如果返回1，则表示成功

#mongodb 的用户名和密码是基于特定数据库的，而不是基于整个系统的。所有所有数据库 db 都需要设置密码
mongo admin -u root -p mongo

#新建一个其它数据库的用户
11.新建库
use foo
12.创建用户密码
db.createUser(
... {
... user: "swen",
... pwd: "swen",
... roles: [ { role: "readWrite", db: "foo" }, ]
... }
... )
13.验证
db.auth("swen", "swen")
14.退出
exit
15.演示往 test集合插入简单数据，并查看数据库状态
db.test.insert({"name":"hello world"})
db.status()

#=================================================================================================
URI 形式的访问
生产中常用 URI 形式对数据库进行连接
mongodb://120.78.3.1:27017/foo

添加用户名密码验证
mongodb://swen:swen@120.78.3.1:27017/foo

spring.data.mongodb.uri=mongodb://name:pass@localhost:27017/test
多个 IP 集群可以采用以下配置：
spring.data.mongodb.uri=mongodb://user:pwd@ip1:port1,ip2:port2/database


一、mongodb概念
1.数据库：
MongoDB的一个实例可以拥有一个或多个相互独立的数据库，每个数据库都有自己的集合。
2.集合：
集合可以看作是拥有动态模式的表。
3.文档：
文档是MongoDB中基本的数据单元，类似关系型数据库表的行。文档是键值对的一个有序集合。
4._id
每个文档都有个特殊的"_id",在文档所属集合中是唯一的。
5.mongo 命令 链接 MongoDB 数据库时 会进入 javascript shell
MongoDB自带了一个功能强大的JavaScript Shell,可以用于管理或操作MongoDB。
6.数据库命名规则
	* 1:不能是空串。
	* 2:不得含有/、\、?、$、空格、空字符等等,基本只能使用ASCII中的字母和数字 。
	* 3:区分大小写,建议全部小写。
	* 4:最多为64字节。
	* 5:不得使用保留的数据库名,比如:admin,local,config 
	注意:数据库最终会成为文件,数据库名就是文件的名称
7.集合名称定义规则
	* 1: 不能是空串。
	* 2: 不能包含 \ 0 字符 (空字符), 这个字符表示集合名的结束, 也不能包含”$”。
	* 3: 不能以”system.” 开头, 这是为系统集合保留的前缀
8.文档的 key 定义规则
	* 1: 不能包含\0字符(空字符),这个字符表示键的结束。
	* 2: “.”和“$”是被保留的,只能在特定环境下用。
	* 3: 区分类型,同时也区分大小写。
	* 4: 键不能重复
	注意:文档的键值对是有顺序的,相同的键值对如果有不同顺序的话,也是不同的文档。

二、数据库操作
1.显示现有的数据库(要显示数据库，需要至少插入一个文档，空的数据库是不显示出来的)
	show dbs
2.显示当前使用的数据
	db
3.切换当前使用的数据库
	use foo
4.创建数据库
    use 数据库名称
    db.集合名称.insert({"name":"wang wu"})
	创建数据库:MongoDB没有专门创建数据库的语句,可以使用“use” 来使用某个数据库,如果要使用
的数据库不存在,那么将会创建一个,会在真正向该库加入文档后,保存成为文件。
5.删除数据库
	use 数据库名称
	db.dropDatabase()

三、集合的操作
1.创建集合
在MongoDB中不用创建集合,因为没有固定的结构,直接使用db.集合名称.命令 来操作就可
以了。如果非要显示创建集合的话,用:db.createCollecion(“集合名称”);
2.显示现有的集合
    use foo
    show collections
3.insert 可以插入一个用 {};多条数据用 []
    db.userdatas.insert([ {"name":'lisan',"age":23},{"name":"wang wu","age":33} ])
    ps:1:MongoDB会为每个没有“_id”字段的文档自动添加一个”_id”字段
	   2:每个Doc必须小于16MB
4.删除文档 db. 集合. remove()
    db.userdatas.remove({"name":"lisan"})
5.查看数据库 文档状态 可以看到文档的个数大小 等等信息
    数据库： db.stats()
    文档：   db.集合.stats()
6.查看集合中所有的文档
    db.集合名称.find();
7.文档替换 
	db.集合名称.update(条件, 新的文档); 只会修改符合条件的第一个文档。
	db.test.update({"age":12},{"address":"北京","name":"老王"})    
8.$set:指定一个字段的值, 如果字段不存在, 会创建一个.
	db.test.update({"name":"u1"},{"$set":{"name":"u2"}},0,1)
9.$unset:删掉某个字段
	db.test.update({"name":"u1"},{"$unset":{"address":1}},0,1)
10.$push:向已有数组的末尾加入一个元素, 要是没有就新建一个数组。
	db.test.update({"name":"u1"},{"$push":{"score":2}},0,1)	
11.$each: 通过一次 $push 来操作多个值
	db.test.update({"name":"u1"},{"$push":{"score":{"$each":[4,5,6]}}})
12.save :如果文档存在就更新, 不存在就新建, 主要根据”_id” 来判断。
	db.test.save({"name":"li si"})
13.update
	第三个参数 0 表示查找不到，不增加一个文档 ;1 表示查找不到，增加文档
	第四个参数 0 表示只更新第一个;1 更新所有查找到的文档
	1: 只能用在 $XXX 的操作中
	2: 最好每次都显示的指定 update 的第 4 个参数, 以防止服务器使用默认行为
14.	find
   14.1
     1)find(条件, 显示的字段)
        db.userdatas.find({},{name:1,_id:0})
     2)比较操作:$lt,$lte,$gt,$gte,$ne
        db.userdatas.find({age:{$gte:35}})
        30<=age<=35
        db.userdatas.find({age:{$lte:35,$gte:30}})
     3)$and: 包含多个条件, 他们之间为 and 的关系
        db.userdatas.find({"name":"u5",age:{$gt:33}})
        db.userdatas.find({$and:[{"name":"u5"},{age:{$gt:33}}]})
     4)$or : 包含多个条件, 他们之间为 or 的关系 ,$nor 相当于 or 取反
        db.userdatas.find({$or:[{"name":"u5"},{age:{$gt:33}}]})   
     5)$not: 用作其他条件之上, 不能作为顶级查询，后面可跟条件，正则
        db.userdatas.find({age:{$not:{$lt:35}}})
        db.userdatas.find({name:{$not:/wang/}})
     6)$mod: 将查询的值除以第一个给定的值, 如果余数等于等二个值则匹配成功。
        db.userdatas.find({age:{$mod:[10,7]}})
        db.userdatas.find({age:{$not:{$mod:[10,7]}}})
     7)$in : 查询一个键的多个值, 只要键匹配其中一个即可 , $nin 为不包含
        db.userdatas.find({age:{$in:[33,35]}})
        db.userdatas.find({age:{$nin:[33,35]}})
     8)$all: 键需要匹配所有的值, 用于数组
        db.userdatas.find({score:{$all:[7,4]}})
     9)$exists: 检查某个键是否存在, 1 表示存在, 0 表示不存在
        db.userdatas.find({"score":{$exists:1}})
     10)null 类型: 不仅能匹配键的值为 null, 还匹配键不存在的情况
        db.userdatas.find({score:null})
        db.userdatas.find({name:{$in:[null],$exists:1}})   
     11)正则
        MongoDB使用Perl兼容的正则表达式(PCRE),
        比如: db.users.find({“name”:/sishuok/i}); 
        又比如: db.users.find({“name”:/^sishuok/});
   14.2 数组搜索
   	  1)单个元素匹配, 就跟前面写条件一样,{key:value}
   		db.userdatas.find({score:0})
	  2)多个元素匹配, 使用 $all, {key:{$all:[a,b]}}, 元素的顺序无所谓
    	db.userdatas.find({score:{$all:[2,0]}})
      3)可以使用索引指定查询数组特定位置, {“key. 索引号”:value}
    	db.userdatas.find({"score.0":7})  索引从0开始
      4)查询某个长度的数组, 使用 $size
    	db.userdatas.find({"score":{$size:4}})
      5)指定子集, 使用 $slice, 正数是前面多少条, 负数是尾部多少条, 也可以指定偏移量和要返回的元素数量
      	比如:$slice:[50,10]
            db.userdatas.find({},{"score":{$slice:2}})
            db.userdatas.find({},{"score":{$slice:-2}})
            db.userdatas.find({},{"score":{$slice:[1,1]}})
 	  6)可以使用 $ 来指定符合条件的任意一个数组元素, 如:{”users.$”:1}
    		db.userdatas.find({"score":{$in:[7,4]}},{"score.$":1,name:1})
      7)$elemMatch: 要求同时使用多个条件语句来对一个数组元素进行比较判断
    		db.userdatas.find({score:{$elemMatch:{$gt:8,$lt:12}}})
   14.3 查询内嵌文档
        1.查询整个内嵌文档与普通查询是一样的 。
        2.如果要指定键值匹配, 可以使用 “.” 操作符, 比如:{“name.first”:”a” ,“name.last”:”b”} 。
        3.如果要正确的指定一组条件, 那就需要使用 $elemMatch, 以实现对内嵌文档的多个键进行匹配操作。
        插入一个文档：
            db.userdatas.update({"name":"u2"},{$set:{wendang:{"yw":80,"xw":90}}})
        查询文档：
             db.userdatas.find({"wendang.yw":80}) 		
   14.4 查询记录条数的命令：count
        1.直接使用 count() 的话，得到的是整个记录的条数。
        2.如果要获取按条件查询后记录的条数，需要指定 count(true 或者非 0 的数)
            db.userdatas.find().count()
            db.userdatas.find().limit(2).count(true)
   14.5 限制返回的记录条数的命令：limit(要返回的条数)
    	db.userdatas.find().limit(2)
   14.6 限制返回的记录条数起点的命令: skip(从第几条开始返回)
    	db.userdatas.find().skip(0).limit(2)
   14.7 排序的命令: sort({要排序的字段: 1 为升序,-1 为降序})
    	db.userdatas.find().sort({age:-1})  
    	db.userdatas.find().sort({age:-1}).limit(2)
   14.8 分页
        分页查询: 组合使用 limit,skipt 和 sort skip 效率低
        当然也可以使用其他方式来分页, 比如采用自定义的 id, 然后根据 id 来分页
   14.9 查询给定键的所有不重复的数据, 命令: distinct
     	db.runCommand({“distinct”:集合名,“key”:”获得不重复数据的字段”});
15.游标
    1: 获取游标, 示例如下:
      var c = db.users.find();
	2: 循环游标, 可以用集合的方式, 示例如下:
       while(c.hasNext()){
             printjson(c.next().文档键);
       } 
	3: 也可以使用 forEach 来循环, 示例如下:
		c.forEach(function(obj){
             print(obj);
        });

mongodb使用场景
1.更高的写入负载
默认情况下，MongoDB 更侧重高数据写入性能，而非事务安全，MongoDB 很适合业务系统中有大量 “低价值” 数据的场景。但是应当避免在高事务安全性的系统中使用 MongoDB，除非能从架构设计上保证事务安全。
(MongoDB 是否支持事务？MongoDB 只支持行级的事务，或者说支持原子性，单行的操作要么全部成功，要么全部失败)
2.高可用性
MongoDB 的复副集 (Master-Slave) 配置非常简洁方便，此外，MongoDB 可以快速响应的处理单节点故障，自动、安全的完成故障转移。这些特性使得 MongoDB 能在一个相对不稳定（如云主机）的环境中，保持高可用性。
3.数据量很大或者未来会变得很大
依赖数据库 (MySQL) 自身的特性，完成数据的扩展是较困难的事，在 MySQL 中，当一个单达表到 5-10GB 时会出现明显的性能降级，此时需要通过数据的水平和垂直拆分、库的拆分完成扩展，使用 MySQL 通常需要借助驱动层或代理层完成这类需求。而 MongoDB 内建了多种数据分片的特性，可以很好的适应大数据量的需求。
4.基于位置的数据查询
MongoDB 支持二维空间索引，因此可以快速及精确的从指定位置获取数据。
5.表结构不明确，且数据在不断变大
在一些传统 RDBMS 中，增加一个字段会锁住整个数据库 / 表，或者在执行一个重负载的请求时会明显造成其它请求的性能降级。通常发生在数据表大于 1G 的时候（当大于 1TB 时更甚）。 因 MongoDB 是文档型数据库，为非结构货的文档增加一个新字段是很快速的操作，并且不会影响到已有数据。另外一个好处当业务数据发生变化时，是将不在需要由 DBA 修改表结构。
6.没有 DBA 支持
如果没有专职的 DBA，并且准备不使用标准的关系型思想（结构化、连接等）来处理数据，那么 MongoDB 将会是你的首选。MongoDB 对于对像数据的存储非常方便，类可以直接序列化成 JSON 存储到 MongoDB 中。 但是需要先了解一些最佳实践，避免当数据变大后，由于文档设计问题而造成的性能缺陷。


应用特征	                                                                               YES / NO
应用不需要事务及复杂 join 支持	                                                            必须 Yes
新应用，需求会变，数据模型无法确定，想快速迭代开发	                                            ?
应用需要 2000-3000 以上的读写 QPS（更高也可以）	                                                ?
应用需要 TB 甚至 PB 级别数据存储	                                                           ?
应用发展迅速，需要能快速水平扩展	                                                            ?
应用要求存储的数据不丢失	                                                                   ?
应用需要 99.999% 高可用	                                                                   ?
应用需要大量的地理位置查询、文本查询	                                                          ?
如果上述有 1 个 Yes，可以考虑 MongoDB，2 个及以上的 Yes，选择 MongoDB 绝不会后悔。
```



###docker mongodb带认证副本集搭建

```shell
#1.开放端口
firewall-cmd --permanent --add-port=8848/tcp
firewall-cmd --permanent --add-port=8849/tcp
firewall-cmd --reload
firewall-cmd --list-all
#2.创建密钥文件(这个密钥文件的所有者被设置成 id 为 “999” 的用户了，因为在 MongoDB 的 Docker 容器中，这个用户需要有操作密钥文件的权限)
cd /apps/svr/database/mongo
openssl rand -base64 741 > mongodb-keyfile
chmod 600 mongodb-keyfile
chown 999 mongodb-keyfile
#3.启动 node1（即第一台 Docker 服务器）的 MongoDB 容器。它会启动一个没有身份验证机制的容器，所以我们要设置一个用户
docker run --name mongo_master -v /apps/svr/database/mongo/data:/data/db -v /apps/svr/database/mongo:/opt/keyfile -p 8848:27017 --restart=always -d docker.io/mongo --smallfiles
#进入mongodb容器
docker exec -it mongo_master /bin/bash
#进入mongodb
mongo
#使用admin数据库
use admin
#创建用户密码
db.createUser({user: "cesc",pwd: "fabregas4",roles: [ { role: "userAdminAnyDatabase", db: "admin" }]});
db.createUser({user: "root",pwd: "root_9527",roles: [ {role: "root",db: "admin"} ]});
#清除日志
db.runCommand({ logRotate :1 } )

#4.退出，停止并删除第一个实例
exit->exit->docker stop mongo_master->docker rm mongo_master

#5.使用密钥文件启动两台 MongoDB 实例
docker run --name mongo_master -v /apps/svr/database/mongo/data:/data/db -v /apps/svr/database/mongo:/opt/keyfile -p 8848:27017 --restart=always -d docker.io/mongo --smallfiles --keyFile /opt/keyfile/mongodb-keyfile --replSet "rs1"

docker run --name mongo_slave -v /apps/svr/database/mongo/data_slave:/data/db -v /apps/svr/database/mongo:/opt/keyfile -p 8849:27017 --restart=always -d docker.io/mongo --smallfiles --keyFile /opt/keyfile/mongodb-keyfile --replSet "rs1"
#进入mongodb容器
docker exec -it mongo_master bash
#进入mongodb
mongo(mongo --port 27017)
#使用admin数据库
use admin
#验证
db.auth("root", "root_9527")

#6.配置节点
myconf = {"_id":"rs1","members":[{"_id":0,"host":"120.78.3.1:8848"},{"_id":1,"host":"120.78.3.1:8849"}]}
#7.初始化配置
rs.initiate(myconf)
#查看当前复制集的节点信息
rs.isMaster()
#查看状态
rs.status()
#查看配置
rs.conf()
#登录从节点
docker exec -it mongo_slave bash
mongo --port 27017
#设置从节点可以读
rs.slaveOk()
#db.getMongo().setSlaveOk();

#参考地址
https://www.jianshu.com/p/7bfda4943034
https://www.cnblogs.com/jinjiangongzuoshi/p/9301062.html
```



### docker redis主从、哨兵搭建

```shell
#一主两从搭建
1.拉取redis镜像
docker pull redis
#开发端口
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --permanent --add-port=6380/tcp
firewall-cmd --permanent --add-port=6378/tcp
2.
分别在/apps/svr/database/redis下创建redis-6379.conf,redis-6378.conf,redis-6380.conf以及各自data目录
内容如下：(一主两从),防火墙模式下开放三个端口
port 6379
logfile "redis-6379.log"
dir /data
appendonly yes
appendfilename appendonly.aof
masterauth admin9527
requirepass admin9527
#=================================================================================================
port 6380
logfile "redis-6380.log"
dir /data
appendonly yes
appendfilename appendonly.aof
slaveof 120.78.3.1 6379
masterauth admin9527
requirepass admin9527
#=================================================================================================
port 6378
logfile "redis-6378.log"
dir /data
appendonly yes
appendfilename appendonly.aof
slaveof 120.78.3.1 6379
masterauth admin9527
requirepass admin9527
3.启动docker容器
docker run -p 6379:6379 --restart=always --name redis_master -v /apps/svr/database/redis/redis-6379.conf:/etc/redis/redis-6379.conf -v /apps/svr/database/redis/redis-6379-data:/data -d docker.io/redis  redis-server /etc/redis/redis-6379.conf

docker run -p 6378:6378 --restart=always --name redis_slave_one -v /apps/svr/database/redis/redis-6378.conf:/etc/redis/redis-6378.conf -v /apps/svr/database/redis/redis-6378-data:/data -d docker.io/redis  redis-server /etc/redis/redis-6378.conf

docker run -p 6380:6380 --restart=always --name redis_slave_two -v /apps/svr/database/redis/redis-6380.conf:/etc/redis/redis-6380.conf -v /apps/svr/database/redis/redis-6380-data:/data -d docker.io/redis  redis-server /etc/redis/redis-6380.conf
4.进入容器查看状态：
docker exec -it redis_master /bin/bash
redis-cli(redis-cli -p 6378;redis-cli -p 6380)
auth admin9527
info replication

#哨兵集群搭建
#1.开发端口
firewall-cmd --permanent --add-port=26379/tcp
firewall-cmd --permanent --add-port=26380/tcp
firewall-cmd --permanent --add-port=26378/tcp
#2.在/apps/svr/database/redis目录下创建如下三个配置文件
port 26379
dir /data
logfile "sentinel-26379.log"
sentinel monitor mymaster 120.78.3.1 6379 2
sentinel down-after-milliseconds mymaster 3000
sentinel failover-timeout mymaster 18000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster admin9527
==================================================================================================
port 26380
dir /data
logfile "sentinel-26380.log"
sentinel monitor mymaster 120.78.3.1 6379  2
sentinel down-after-milliseconds mymaster 3000
sentinel failover-timeout mymaster 18000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster admin9527
==================================================================================================
port 26378
dir /data
logfile "sentinel-26378.log"
sentinel monitor mymaster 120.78.3.1 6379  2
sentinel down-after-milliseconds mymaster 3000
sentinel failover-timeout mymaster 18000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster admin9527

3.启动容器
docker run -p 26379:26379 --restart=always --name sentinel_one -v /apps/svr/database/redis/sentinel-26379.conf:/etc/redis/sentinel.conf -v /apps/svr/database/redis/sentinel-26379-data:/data -d docker.io/redis redis-sentinel /etc/redis/sentinel.conf

docker run -p 26378:26378 --restart=always --name sentinel_two -v /apps/svr/database/redis/sentinel-26378.conf:/etc/redis/sentinel.conf -v /apps/svr/database/redis/sentinel-26378-data:/data -d docker.io/redis redis-sentinel /etc/redis/sentinel.conf

docker run -p 26380:26380 --restart=always --name sentinel_three -v /apps/svr/database/redis/sentinel-26380.conf:/etc/redis/sentinel.conf -v /apps/svr/database/redis/sentinel-26380-data:/data -d docker.io/redis redis-sentinel /etc/redis/sentinel.conf

4.进入容器
docker exec -it sentinel_one /bin/bash
5.进入redis
redis-cli -p 26379
6.验证信息
info sentinel
```



```shell
#docker 安装
yum  install docker -y
#查看版本
docker -v
#启动
service docker start/systemctl start docker
#守护进程重启   
sudo systemctl daemon-reload
#重启 docker 服务   
systemctl restart  docker
#重启 docker 服务  
sudo service docker restart
#关闭 docker   
service docker stop
#关闭 docker  
systemctl stop docker

#查看镜像
docker images
#删除镜像
docker rmi IMAGEID 
镜像操作：
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    images    List images
    load      Load an image from a tar archive or STDIN
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rmi       Remove one or more images
    search    Search the Docker Hub for images
    tag       Tag an image into a repository
    save      Save one or more images to a tar archive 
    history   显示某镜像的历史
    inspect   获取镜像的详细信息

# 查看运行中的容器
docker ps
# 查看所有容器
docker ps -a
# 退出容器
按Ctrl+D 即可退出当前容器【但退出后会停止容器】
# 退出不停止容器：
组合键：Ctrl+P+Q
# 启动容器
docker start 容器名或ID
# 进入容器
docker attach 容器名或ID
# 停止容器
docker stop 容器名或ID
# 暂停容器
docker pause 容器名或ID
# 继续容器
docker unpause 容器名或ID
# 删除容器
docker rm 容器名或ID
# 删除全部容器--慎用
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
# 保存容器，生成镜像
docker commit 容器ID 镜像名称

# 从 host 拷贝文件到 container 里面
docker cp /home/soft centos:/webapp
#从主机复制到容器
sudo docker cp host_path containerID:container_path
#从容器复制到主机
sudo docker cp containerID:container_path host_path


# docker 列出每个容器的IP
docker inspect -f='{{.Name}} {{.NetworkSettings.IPAddress}} {{.HostConfig.PortBindings}}' $(docker ps -aq)
# 空间清理
docker system prune
```

### docker 制作 tomcat镜像

```shell
#1.安装 Centos 镜像
docker pull daocloud.io/library/centos:latest
#2.自定义 Tomcat/Jdk 镜像:因为不同项目对 tomcat、jdk 的版本要求不同，docker 提供使用 Dockerfile 来定制镜像，首先创建一个干净的目录 tomcat9_jdk8
mkdir tomcat9_jdk8
#3.编辑 Dockerfile 文件
FROM        daocloud.io/library/centos:latest
MAINTAINER    cescforz@foxmail.com

#把java与tomcat添加到容器中
COPY tomcat9  /apps/svr/tomcat9/
COPY jdk1.8  /apps/svr/jdk1.8/

#配置java与tomcat环境变量
ENV JAVA_HOME /apps/svr/jdk1.8
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /apps/svr/tomcat9
ENV CATALINA_BASE /apps/svr/tomcat9
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

#容器运行时监听的端口
EXPOSE  8080
CMD /apps/svr/tomcat8/bin/catalina.sh run

#4.编辑完成后，使用 docker build -t tomcat9:1.0  命令生成镜像 -t 指定 image 的 tags ， 注意该命令后面的点 （.）指当前文件
docker build -t cesc/tomcat9:1.0 .

#5.启动容器
docker run -d -p 8089:8080 -v /apps/svr/tomcat9_jdk8/tomcat9:/apps/svr/tomcat9 -v /etc/localtime:/etc/localtime --restart=always --name tomcat_foo cesc/tomcat9:1.0

#6.进入容器
docker exec -it tomcat_foo /bin/bash

#7.查看日志
docker logs --tail=100 -f CONTAINER_NAME
```











```shell
#docker安装命令
apt-get update
#vim
apt-get install -y vim
#ps
apt-get install procps
#telnet
apt-get install  telnet 
#ifconfig
apt-get install  net-tools
#ping
apt-get install iputils-ping
```



防火墙命令

```shell
一、iptables 防火墙
1、基本操作
# 查看防火墙状态
service iptables status(/bin/systemctl status iptables.service)
# 停止防火墙
service iptables stop(/bin/systemctl stop iptables.service)
# 启动防火墙
service iptables start  
# 重启防火墙
service iptables restart  
# 永久关闭防火墙
chkconfig iptables off  
# 永久关闭后重启
chkconfig iptables on　　
2、开启 80 端口
vim /etc/sysconfig/iptables
# 加入如下代码
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
保存退出后重启防火墙
systemctl restart iptables.service
==================================================================================================
二、firewall 防火墙
1、查看 firewall 服务状态
systemctl status firewalld
出现 Active: active (running) 切高亮显示则表示是启动状态。
出现 Active: inactive (dead) 灰色表示停止，看单词也行。
2、查看 firewall 的状态
firewall-cmd --state
3、开启、重启、关闭、firewalld.service 服务
# 开启
service firewalld start(/bin/systemctl start firewalld.service)
# 重启
service firewalld restart
# 关闭
service firewalld stop(/bin/systemctl stop firewalld.service)
4、查看防火墙规则
firewall-cmd --list-all 
5、查询、开放、关闭端口
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
1、firwall-cmd：是 Linux 提供的操作 firewall 的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```

