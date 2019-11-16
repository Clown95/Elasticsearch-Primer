---

title: ElasticSearch(二) 环境搭建
tags: Elastic Stack
author: Clown95

---

# ElasticSearch环境搭建
>我们使用的环境centos7以及elasticsearch6.1.1

1. 我们将禁用 CentOS 7 服务器上的 SELinux。 编辑 SELinux 配置文件。
	```bash
	sudo vim /etc/sysconfig/selinux
	```
2. 将 SELINUX 的值从` enforcing `改成` disabled`，改完重启系统。

3. 接着我们更换阿里云软件源
	```bash
	sudo curl -o /etc/yum.repos.d/CentOS-Base.repo http:   mirrors.aliyun.com/repo/Centos-7.repo
	```
4. 更换完成后，我们清除下yum缓存
	```bash
	yum clean all
	```

##  安装JDK
### 删除OpenJDK

CentOS系统自带OpenJDK,使用命令查看
```bash
java -version
```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558314153803.png)

OpenJDK也能用于环境配置，如果不想折腾的可以用OpenJDK,但是官方建议安装Oracle的JDK8.

1. 首先我们使用命令查看 OpenJDK的相关文件：
	```bash
	rpm -qa | grep java
	```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558314610158.png)

2. 接着我们使用命令删除这些文件：
	```bash
	rpm -e --nodeps `rpm -qa | grep java`
	```
>直接输入命令可能会提示权限不够，我们可以使用sudo 或者使用root账号来执行命令。

3. 命令完成后我们再次使用` java -version` 来验证下是否删除。

### 安装JDK8

1. 从Oracle 下载JDK1.8

	Oracal账号(仅用于JDK下载)
	用户名：541509124@qq.com
	密码：LR4ever.1314

2. 使用`rmp` 命令安装
	```bash
	sudo rpm -ivh jdk-8u211-linux-x64.rpm
	```

3. 最后，检查 java JDK 版本，确保它正常工作

	```bash
	java -version
	```

## 安装Elasticsearch
### Elasticsearch安装

1. 在安装 Elasticsearch 之前，将 elastic.co 的密钥添加到服务器。
	```bash
	sudo rpm --import https:   artifacts.elastic.co/GPG-KEY-elasticsearch
	```
	如果执行上面的命令出现`Peer reports incompatible or unsupported protocol version` 这种错误。
	先执行下下面的命令,然后再添加elastic.co 的密钥。
	```bash
	sudo yum update -y nss curl libcurl
	```
2. 接下来，使用 `wget`下载 Elasticsearch 6.1.1.
	```bash
	wget https:   artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.1.rpm
	```
3. 然后使用 `rpm` 安装它
	```bash
	sudo rpm -ivh elasticsearch-6.1.1.rpm
	```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558350735049.png)
### Elasticsearch文件结构
Elasticsearch 已经安装好了，下面我们简单的看下es目录结构：
```markdown
elasticsearch                     -- path.home, es的安装目录
├─bin                             -- ${path.home}/bin, 启动脚本方式目录
├─config                          -- ${path.home}/config, 配置文件目录
├─data                            -- ${path.home}/data, 数据存放目录
│  └─elasticsearch                -- ${path.home}/data/${cluster.name}
├─lib                             -- ${path.home}/lib, 运行程序目录
├─logs                            -- ${path.home}/logs, log目录 
└─plugins                         -- ${path.home}/plugins, 插件目录
    ├─head
    │  └─...
    └─marvel
        └─...
```
### Elasticsearch 配置文件
Elasticsearch配置文件位于config目录中
1. 进入配置目录编辑 `elasticsaerch.yml` 配置文件。
	```bash
	sudo vim /etc/elasticsearch/elasticsearch.yml
	```
2. 在 Network 块中，取消注释 network.host 和 http.port 行，并且修改
`192.168.0.1` 为`0.0.0.0` ，方便我们可以外网访问，
	```bash
	network.host: 0.0.0.0
	http.port: 9200
	```
> 注意： 改成`0.0.0.0` 是很愚蠢的行为，因为这样很不安全，最好改成自己Centos的外网IP

![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558351884522.png)

3. 使用`wq!` 保存文件并退出编辑器。Elasticsearch 配置到此结束。Elasticsearch 将在本机的 9200 端口运行，

### 其他配置文件说明

**elasticsearch.yml es的相关配置**
    - cluster.name 集群名称,以此作为是否同一集群的判断条件 
    - node.name 节点名称,以此作为集群中不同节点的区分条件
    - network.host / http.port 网络地址和端口,用于http和transport服务使用
    - path.data 数据存储地址
    - path.log 日志存储地址

> 同一集群的节点cluster.name 必须一样。

 **jvm.options   jvm的相关参数**
`jvm.options` 主要是配置JVM相关的东西，我们最主要的就是配置`heap`的大小,配置默认的是2g。
```bash
-Xms2g
-Xmx2g
```
可能有的同学可能笔记本或者虚拟机的内存不大，但是配置文件中`heap`分配的比较大，会照成`Eelasticsearch`运行不起来的情况，这时候我们只需要把heap改小就行。 

### 检查Elasticsearch并设置开机启动
1. 重新加载 systemd，将 Elasticsearch 置为开机启动.
	```bash
	sudo systemctl daemon-reload
	sudo systemctl enable elasticsearch
	```
2. 接着我们启动Eelasticsearch
	```bash
	sudo systemctl start elasticsearch
	```
3. 等待 Eelasticsearch 启动成功，然后检查服务器上打开的端口，确保 9200 端口的状态是`LISTEN`
	```bash
	netstat -plntu
	```
4. 如果9200端口没有被监听，使用下面命令查看 es 状态，查找错误。
	```bash
	systemctl status elasticsearch -l
	```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558352301666.png)

5. 使用`curl` 命令检查是否出现json文件，如果出现即代表启动成功  
```bash
curl -X GET http://localhost:9200
```
## 安装Kibana
### 下载并安装Kibana
1. 用 `wget`下载 Kibana 6.1.1
	```bash
	wget https:   artifacts.elastic.co/downloads/kibana/kibana-6.1.1-x86_64.rpm
	```
2. 然后使用 `rpm `命令安装：
	```bash
	sudo rpm -ivh kibana-6.1.1-x86_64.rpm
	```
### 配置Kibana
1. 进入配置目录编辑 `kibana.yml` 配置文件
	```bash
	sudo vim /etc/kibana/kibana.yml
	```
2. 去掉配置文件中` server.port`、`server.host` 和` elasticsearch.url` 这三行的注释。
	```markdown
	server.port: 5601
	server.host: "localhost"
	elasticsearch.url: "http://localhost:9200"
	```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558630284898.png)
3. 保存并退出。

### 检查Kibana并设置开机启动
1. 将 Kibana 设为开机启动，并且启动 Kibana 。
	```bash
	sudo systemctl enable kibana
	sudo systemctl start kibana
	```
2. 检查 Kibana是否运行在端口 5601 上。
	```bash
	netstat -plntu
	```
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558630559367.png)

3. 浏览器输入`http://localhost:5601`检查安装
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1559207820605.png)
 
## 安装中文分词器
常见的中文分词系统有`IK` 和`jieba`，这里我们选择`IK` ,具体的介绍我们后面再说。

1. 首先我们使用` elasticsearch-plugin` 来安装`IK`  。
	```bash
	sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.1.1/elasticsearch-analysis-ik-6.1.1.zip
	```
> 注意： IK的版本要和ES的版本完全匹配
> `/usr/share/elasticsearch` 是你ES所在的安装目录，请根据自己的情况调整。

2. 最终终端出现`Installed analysis-ik`,代表安装成功。

3. 安装完毕后我们必须要重新启动下ES
	```bash
	sudo systemctl restart elasticsearch
	```
## 附加：配置chrome+sense插件 (可替代kibana)

因为我们已经给es配置了外网访问，这时候我们就可以使用我们自己的生产环境来使用es。 这里我给大家推荐一个组合 chrome +sense插件

1. 我们首先下载 sense 插件
	```
	链接:https://pan.baidu.com/s/1vbgfBH5Cv6JzXRdpctO0nA  密码:rc7m
	```
2. 解压文件sense文件
3. 进入`sense`目录，找到`index.html`，修改`localhost` 为你es环境的外网IP
	```html
	 <input id="es_server" type="text" class="span5" value="localhost:9200"/>
	```
4. 进入扩展程序中心，启用开发者模式，点击`加载已解压的扩展程序`，选择刚才的文件夹就行了。

5. 这时候我们打开`chrome`浏览器，点击`sense` 可以看到以下界面
![](https://www.github.com/Clown95/StroyBack/raw/master/小书匠/1558798046032.png)


