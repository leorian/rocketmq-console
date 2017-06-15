# <center>RocketMQ集群部署</center> #
<hr/>
<h3>1、官网下载rocket-mq安装包</h3>

  官网地址：
> <br/>
> http://rocketmq.incubator.apache.org

  linux命令行下载安装包解压：
> <br/>
> <span style="color:green">wget http://mirror.bit.edu.cn/apache/incubator/rocketmq/4.0.0-incubating/rocketmq-all-4.0.0-incubating-bin-release.zip</span>
> <br/>
> <span style="color:green">unzip -q rocketmq-all-4.0.0-incubating-bin-release.zip</span>

  具体操作步骤截图

  ![](http://i.imgur.com/MGZ3Xve.png)
  ![](http://i.imgur.com/hCSJdGa.png)

<h3>2、rocketmq集群部署示意图</h3>
注意：由于机器有限，示意图展示的是双Master/Slave模式集群部署方案示意图,本文档记录的双Master模式

![](http://i.imgur.com/Wh5jnUP.png)

两台NameServer集群清单
<table>
	<caption>NameServer集群</caption>
	<tr>
	   <th>IP</th>
	   <th>PORT</th>
	   <th>作用</th>
	   <th>启动命令</th>
	</tr>
	<tr>
		<td>172.16.150.143</td>
		<td>9876(默认端口)</td>
		<td>name server 1</td>
		<td> nohup sh bin/mqnamesrv &(参考)</td>
	</tr>
	<tr>
		<td>172.16.150.178</td>
		<td>9876(默认端口)</td>
		<td>name server 2</td>
		<td> nohup sh bin/mqnamesrv &(参考)</td>
	</tr>
</table>

两台BrokerServer集群清单
<table>
	<caption>BrokerServer集群</caption>
	<tr>
		<th>IP</th>
		<th>PORT</th>
		<th>模式</th>
		<th>名称</th>
		<th>ID</th>
		<th>启动命令</th>
	</tr>
	<tr>
		<td>172.16.150.143</td>
		<td>10911(默认端口)</td>
		<td>Master</td>
		<td>broker-a</td>
		<td>0</td>
		<td>nohup sh bin/mqbroker -c 配置文件 &(参考)</td>
	</tr>
	<tr>
		<td>x.x.x.x</td>
		<td>10911(默认端口)</td>
		<td>Slave</td>
		<td>broker-a</td>
		<td>1(非零值)</td>
		<td>nohup sh bin/mqbroker -c 配置文件 &(参考)</td>
	</tr>
	<tr>
		<td>172.16.150.178</td>
		<td>10911(默认端口)</td>
		<td>Master</td>
		<td>broker-b</td>
		<td>0</td>
		<td>nohup sh bin/mqbroker -c 配置文件 &(参考)</td>
	</tr>
	<tr>
		<td>x.x.x.x</td>
		<td>10911(默认端口)</td>
		<td>Slave</td>
		<td>broker-b</td>
		<td>1(非零值)</td>
		<td>nohup sh bin/mqbroker -c 配置文件 &(参考)</td>
	</tr>
</table>

<h3>3、开始部署</h3>
<h4>(1)、分别启动两台机器上nameserver服务</h4>
172.16.150.143/172.16.150.178

> [root@localhost rocketmq]#<span style="color:green"> pwd</span>
> <br/>
> /opt/rocketmq
> <br/>
> [root@localhost rocketmq]# <span style="color:green">ls</span>
> <br/>
> apache-rocketmq-all  rocketmq-all-4.0.0-incubating-bin-release.zip
> <br/>
> [root@localhost rocketmq]# <span style="color:green">cd apache-rocketmq-all/</span>
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">ls</span>
> <br/>
> benchmark  bin  conf  DISCLAIMER  lib  LICENSE  NOTICE  README.md
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">jps</span>
> <br/>
> 13231 Jps
> <br/>
> 28863 jar
> <br/>
> 31313 Bootstrap
> <br/>
> 31278 Bootstrap
> <br/>
> [root@localhost apache-rocketmq-all]#<span style="color:green"> nohup sh bin/mqnamesrv &</span>
> <br/>
> [1] 13245
> <br/>
> [root@localhost apache-rocketmq-all]# nohup: 忽略输入并把输出追加到"nohup.out"
> <br/>
> ^C
> <br/>
> [root@localhost apache-rocketmq-all]#<span style="color:green"> jps</span>
> <br/>
> 28863 jar
> <br/>
> 31313 Bootstrap
> <br/>
> 13251 NamesrvStartup
> <br/>
> 13300 Jps
> <br/>
> 31278 Bootstrap
> <br/>
> [root@localhost apache-rocketmq-all]# 
> <br/>

<h4>(2)、根据nameserver集群地址分别启动两台机器上brokerserver服务</h4>
<h5 style="color:blue">172.16.150.143</h5>
<br/>
> 编辑配置文件
> <br/>
> /opt/rocketmq/apache-rocketmq-all/conf/2m-noslave/broker-a.properties
> <br/>
> 添加配置项
> <br/>
> namesrvAddr=172.16.150.143:9876;172.16.150.178:9876
> <br/>
> brokerIP1=172.16.150.143

最终内容如下
> <br/>
> brokerClusterName=DefaultCluster
> <br/>
> brokerName=broker-a
> <br/>
> brokerId=0
> <br/>
> deleteWhen=04
> <br/>
> fileReservedTime=48
> <br/>
> brokerRole=ASYNC_MASTER
> <br/>
> flushDiskType=ASYNC_FLUSH
> <br/>
> namesrvAddr=172.16.150.143:9876;172.16.150.178:9876
> <br/>
> brokerIP1=172.16.150.143
> <br/>

启动命令
> [root@localhost apache-rocketmq-all]# <span style="color:green">ls</span>
> <br/>
> benchmark  bin  conf  DISCLAIMER  lib  LICENSE  nohup.out  NOTICE  README.md
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">tail -f conf/2m-noslave/broker-a.properties</span> 
> <br/>
> 
> brokerClusterName=DefaultCluster
> <br/>
> brokerName=broker-a
> <br/>
> brokerId=0
> <br/>
> deleteWhen=04
> <br/>
> fileReservedTime=48
> <br/>
> brokerRole=ASYNC_MASTER
> <br/>
> flushDiskType=ASYNC_FLUSH
> <br/>
> namesrvAddr=172.16.150.143:9876;172.16.150.178:9876
> <br/>
> brokerIP1=172.16.150.143
> <br/>
> ^C
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">nohup sh bin/mqbroker -c /opt/rocketmq/apache-rocketmq-all/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 &</span>
> <br/>
> [1] 9514
> <br/>
> [root@localhost apache-rocketmq-all]#<span style="color:green"> jps</span>
> <br/>
> 9521 BrokerStartup
> <br/>
> 31684 Bootstrap
> <br/>
> 9304 NamesrvStartup
> <br/>
> 9579 Jps
> <br/>
> 31391 Bootstrap
> <br/>
> [root@localhost apache-rocketmq-all]# 
> <br/>


<h5 style="color:blue">172.16.150.178</h5>
<br/>
> 编辑配置文件
> <br/>
> /opt/rocketmq/apache-rocketmq-all/conf/2m-noslave/broker-b.properties
> <br/>
> 添加配置项
> <br/>
> namesrvAddr=172.16.150.143:9876;172.16.150.178:9876
> <br/>
> brokerIP1=172.16.150.178

最终内容如下
> <br/>
> brokerClusterName=DefaultCluster
> <br/>
> brokerName=broker-b
> <br/>
> brokerId=0
> <br/>
> deleteWhen=04
> <br/>
> fileReservedTime=48
> <br/>
> brokerRole=ASYNC_MASTER
> <br/>
> flushDiskType=ASYNC_FLUSH
> <br/>
> namesrvAddr=172.16.150.143:9876;172.16.150.178:9876
> <br/>
> brokerIP1=172.16.150.178
> <br/>

启动命令
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">ls</span>
> <br/>
> benchmark  bin  conf  DISCLAIMER  lib  LICENSE  nohup.out  NOTICE  README.md
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">tail -f conf/2m-noslave/broker-b.properties </span>
> <br/>
> brokerClusterName=DefaultCluster
> <br/>
> brokerName=broker-b
> <br/>
> brokerId=0
> <br/>
> deleteWhen=04
> <br/>
> fileReservedTime=48
> <br/>
> brokerRole=ASYNC_MASTER
> <br/>
> flushDiskType=ASYNC_FLUSH
> <br/>
> namesrvAddr=172.16.150.143:9876;172.16.150.178:9876
> <br/>
> brokerIP1=172.16.150.178
> <br/>
> ^C
> <br/>
> [root@localhost apache-rocketmq-all]# <span style="color:green">nohup sh bin/mqbroker -c /opt/rocketmq/apache-rocketmq-all/conf/2m-noslave/broker-b.properties >/dev/null 2>&1 &</span>
> <br/>
> [2] 14369
> <br/>
> [root@localhost apache-rocketmq-all]#<span style="color:green"> jps</span>
> <br/>
> 28863 jar
> <br/>
> 13732 NamesrvStartup
> <br/>
> 14451 Jps
> <br/>
> 31313 Bootstrap
> <br/>
> 14376 BrokerStartup
> <br/>
> 31278 Bootstrap
> <br/>
> [root@localhost apache-rocketmq-all]# 

<h3>4、启动RocketMQ集群管理平台</h3>

github地址：https://github.com/leorian/rocketmq-console

项目架构是SpringBoot+Angular,只需修改src/main/resources/application.properties文件

修改配置项rocketmq.config.namesrvAddr=172.16.150.143:9876;172.16.150.178:9876

![](http://i.imgur.com/owaiFUd.png)

打包发布服务器 

mvn clean package -Dmaven.test.skip=true

nohup sh java -jar target/rocketmq-console-ng-1.0.0.jar &

![](http://i.imgur.com/zRHlhKu.png)

<h3>5、参考文档</h3>
CSDN-博客
<br/>
http://blog.csdn.net/lovesomnus/article/details/51769977
