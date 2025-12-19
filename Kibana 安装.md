# Kibana 安装

## 概述

Kibana是基于TS语言开发，运行在Node.js环境的一款可视化工具，可以通过Kibana管理、监控、保护Elastic Stack、分析数据、搜索、查看、保护数据

## 在开始本文操作之前，有以下几点需要声明
- **本文中的操作系统为oracle linux 9.6，兼容centos、redhat**  
- **本文假设已安装好操作系统，包括时钟、主机名等操作**  
- **放开了默认SSH 22端口，可远程登陆**  
- **本文采用二进制压缩包的方式安装**  
- **本文操作系统采用X86_64架构**  
- **本文Kibana版本：8.19.8**
- **Kiana与Elasticsearch须完全一致**
- **Kiana不建议使用root启动**
- **操作系统防火墙如打开，可将端口放开**
- **kibana启动进程是node**

## 开始正文

操作系统配置调整
```
下载Elasticsearch，并解压
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.19.8-linux-x86_64.tar.gz
tar -zxvf kibana-8.19.8-linux-x86_64.tar.gz -C /opt/

创建Kibana用户、密码
useradd elk
passwd elk

设置目录权限
chown -R elk:elk /opt/elasticsearch-8.19.8

切换用户
su - elk
```

编辑kibana.yml
```
server.port: 5601

server.host: "192.168.1.30"

server.name: "kibana-1"

elasticsearch.hosts: ["http://192.168.1.30:9200"]

i18n.locale: "zh-CN"

logging.appenders.default:
  type: rolling-file
  fileName: /opt/kibana-8.19.8/logs/kibana.log
  policy:
    type: size-limit
    size: 256mb  # 每个文件最大256MB
  strategy:
    type: numeric
    max: 10      # 保留10个备份文件
  layout:
    type: json
	
# 添加kibana内置用户
elasticsearch.username: kibana_system
elasticsearch.password: kibana-1
```

编辑node.options
```
# 注销旧版本openssl
# --openssl-legacy-provider
```

启动kibana，浏览器确认
```
su - elk
nohup ./kibana &
http://192.168.1.5:5601/
```

##### 至此，Kibana单机版安装就介绍完成了，请开始你的表演吧！

##### 参考链接：https://www.elastic.co/docs/reference/kibana

&nbsp;

**有兴趣的小伙伴，可加联系方式：Telegram：@dean_code**  
