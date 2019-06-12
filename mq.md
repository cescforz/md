```
停止 broker
sh bin/mqshutdown broker
停止 namesrv
sh bin/mqshutdown namesrv

启动看日志
tail -f ~/logs/rocketmqlogs/namesrv.log
tail -f ~/logs/rocketmqlogs/broker.log



步骤一，启动 Name Server(进入/apps/svr/apache-rocketmq目录下)
nohup sh bin/mqnamesrv > /dev/null 2>&1 & tail -f ~/logs/rocketmqlogs/namesrv.log
==================================================================================================
步骤二，指定 Broker 外网IP
添加
vi /opt/apache-rocketmq/conf/broker.conf
brokerIP1=47.106.96.225
输入终端执行
export NAMESRV_ADDR=47.106.96.225:9876
==================================================================================================
步骤三，启动 Broker
nohup sh bin/mqbroker -n 47.106.96.225:9876 > autoCreateTopicEnable=true -c conf/broker.conf /dev/null 2>&1 & tail -f ~/logs/rocketmqlogs/broker.log

步骤四，启动监控页面(进入/apps/svr/rocketmq-externals/rocketmq-console目录下)
nohup java -jar target/rocketmq-console-ng-1.0.0.jar --rocketmq.config.namesrvAddr=47.106.96.225:9876  > /dev/null 2>&1 &
```

