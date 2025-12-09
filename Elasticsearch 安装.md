# Elasticsearch 安装

## 在开始本文操作之前，有以下几点需要声明
- **本文中的操作系统为oracle linux 9.6，兼容centos、redhat**  
- **本文假设已安装好操作系统，包括时钟、主机名等操作**  
- **放开了默认SSH 22端口，可远程登陆**  
- **本文采用二进制压缩包的方式安装**  
- **本文操作系统采用X86_64架构**  
- **本文Elasticsearch版本：8.19.8**
- **Elasticsearch 7以上内置OpenJDK**
- **Elasticsearch与其它组件版本须完全一致**
- **Elasticsearch访问端口9200，内部通信端口9300**
- **Elasticsearch不支持使用root启动**
- **Elasticsearch默认使用G1GC并行收集器，早期版本使用CMS并行收集器**
- **JVM Xms、Xmx值应相同，否则大小调整时会出现停顿**
- **OpenJDK分为客户端JVM和服务端JVM，Elasticsearch默认使用"server JVM"启动**
- **操作系统防火墙如打开，可将端口放开**

## 开始正文

操作系统配置调整
```
# 文件句柄数
echo "elasticsearch  -  nofile  65535" >> /etc/security/limits.conf
# ES 线程数
echo "elasticsearch  -  nproc  8192" >> /etc/security/limits.conf
# 虚拟内存数(as：address space)
echo "elasticsearch  -  as     unlimited" >> /etc/security/limits.conf
# ES 最大文件大小
echo "elasticsearch  -  fsize  unlimited" >> /etc/security/limits.conf

# TCP 重传次数，默认：15次
echo "net.ipv4.tcp_retries2=5" >> /etc/sysctl.conf
# 禁用交换分区
echo "vm.swappiness=1" >> /etc/sysctl.conf
# mmapfs 文件系统数
echo "vm.max_map_count=262144" >> /etc/sysctl.conf

# 重启系统
reboot

创建ES用户、密码
useradd elk
passwd elk

设置目录权限
chown -R elk:elk /opt/elasticsearch-8.19.8

切换用户，并创建目录、临时目录
su - elk
mkdir -p data tmp

指定JNA(Java Native Access)、libffi库的临时目录
echo "export ES_TMPDIR=/opt/elasticsearch-8.19.8/tmp" >> /home/elk/.bash_profile
source /home/elk/.bash_profile
```

下载Elasticsearch，并解压
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.19.8-linux-x86_64.tar.gz
tar -xvf elasticsearch-8.19.8-linux-x86_64.tar.gz -C /opt/
```

设置 elasticsearch.yml
```
# 设置单机模式
discovery.type: single-node
# 设置主机地址
network.host: 192.168.1.30
# 禁用自动创建索引，只允许下面这些索引
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
# 设置数据存放路径
path.data: /opt/elasticsearch-8.19.8
# 设置日志存放路径
path.logs: /opt/elasticsearch-8.19.8/logs 
# ES 名称
node.name: elk-1
# 关闭安全验证
xpack.security.enabled: false
```

设置 JVM配置(jvm.options文件)
```
# 调整JVM大小
-Xms4g
-Xmx4g
```

后台启动
```
elasticsearch -d
```

确认是否正常
```
http://192.168.1.30:9200
```

##### 至此，Elasticsearch单机版安装就介绍完成了，请开始你的表演吧！

&nbsp;

**有兴趣的小伙伴，可加联系方式：Vx：+86 183 1026 3857&nbsp;&nbsp;&nbsp;&nbsp;Telegram：+86 189 1127 2119**  
