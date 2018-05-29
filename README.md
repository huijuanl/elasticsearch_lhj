this is my es_cluster projects.

所用的elasticsearch版本为5.4.1，es官网中可下载,或者该目录下的es zip包也可解压之后直接使用.


head插件安装(head运行依赖于nodejs,npm和grunt)
------------------

在5.x版本中head不是在es中默认安装，而是作为一个插件，head需要去官网下载: https://github.com/mobz/elasticsearch-head.安装步骤请参考https://blog.csdn.net/LLF_1241352445/article/details/77800046

head的安装还依赖nodejs和npm,grunt，安装后用nodejs -v,npm-v和grunt -version来验证安装成功
(命令grunt -version回车之后要出现

grunt-cli 版本号

grunt 版本号才可以)         


elasticesearch集群配置(以一个集群中有三个节点为例)
------------------

1.将官网下载的elasticsearch5.4.1解压，将解压后的文件夹复制两份，然后重命名，每个文件夹代表一个节点的配置

2.修改每文件夹中节点的配置，在$ES_HOME/config/elasticsearch.yml中修改，如:elasticsearch5.4.1/config/elasticsearch.yml

 其中主要设置以下几个参数:
 cluster.name: 集群名（同一集群下的节点中的cluster.name必须相同）
 node.name: 节点名(唯一ID)
 node.master: true
 node.data: false
 http.port: 端口号(如9201,可用于http网页显示)
 transport.tcp.port:端口号(如 9202,这个的设置是节点之间互相ping通互相发现的前提)
 discovery.zen.ping_timeout:
 client.transport.ping_timeout:
 discovery.zen.minimum_master_nodes: 2 
 (es官方认为这个discovery.zen.minimum_master_nodes的值应该是(总共的节点数/2+1),比如我们这里是3个节点，那么就是3/2+1=2)
 discovery.zen.ping.unicast.hosts:["hostip1:transport.tcp.port","hostip2:transport.tcp.port",...]
 (discovery.zen.ping.unicast.hosts: ["127.0.0.1:9202", "127.0.0.1:9302","127.0.0.1:9402"])
 http.cors.enabled:
 http.cors.allow-origin：

 具体设置请参考esnodes/目录下各个节点的设置
3.将es节点启动时所占用的jvm内存重新设置，默认的2g太大，在$ES_HOME/config/jvm.options中修改-Xms和-Xmx：
设置为:
```
-Xms512m
-Xmx512m
```

head配置修改
--------------------
head中修改Gruntfile.js和head/_site/app.js


Gruntfile.js在connect:中增加一行 hostname: '*',
如下所示:

```
connect: {
                        server: {
                                options: {
                                        port: 9100,
                                        hostname: '*',
                                        base: '.',
                                        keepalive: true
                                }
                        }
                }
```

app.js中在this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") 后面增加每个节点的ip:http端口
如下图所示:
```
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9201"||"http://localhost:9301"||"http://localhost:9401";
```

上述配置完毕之后，执行start.sh脚本

若不报错，说明配置成功，执行完一段时间后，打开浏览器，

输入localhost:9100

若页面有三个节点显示，且集群处于健康状态，说明启动成功.

