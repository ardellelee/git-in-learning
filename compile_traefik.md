# 第一什么事traefik

traefik是一个go语言编写的反向代理服务器

大体的作用如下

<img src="pic/architecture.png"/>

举例如下：

用户访问www.baidu.com（此时我们称www.baidu.com为 前向路径 Frontend）从用户来的报文会携带www.baidu.com为目的的信息,报文到达traefik反向代理服务器的时候,
traefik会对前向路径做路径匹配,将前向流量按照匹配模式分配到后台的服务器上去,后端服务器称之为 后向路径 Backend.

假设存在三个用户 地址分别为 x.x.1.x x.x.2.x  x.x.3.x

前向路径Frontend为 www.baidu.com map.baidu.com
后向路径Backend为 192.168.1.56 192.168.1.57 192.168.1.58

Backend 192.168.1.56服务器9110端口运行了www.baidu.com的服务程序
Backend 192.168.1.58服务器9110端口运行了map.baidu.com的服务程序
Backend 192.168.1.57服务器9111端口运行了www.baidu.com以及服务器9112端口map.baidu.com的两个服务程序

traefik代理匹配模式通过访问者IP域来区分匹配

代理流程就是

1、x.x.1.x 访问 www.baidu.com 
2、traefik根据 Frontend（www.baidu.com）并匹配IP段模式 （x.x.1.x）认为 Backend （192.168.1.57服务器9111端口）应该服务
3、traefik将192.168.1.57服务器9111端口暴露给用户建立起服务链接完成代理过程

# traefik获取代理配置信息

traefik可以通过多个数据来源获取代理配置信息 每个配置信息来源称之为一个provider 
配置信息可以写在配置文件traefik.toml 此时称为File provider
也可以保存在Etcd consul等一致性键值数据库中

配置格式如下
```
# traefik.toml
logLevel = "DEBUG"

# 这是traefik状态服务器服务地址
[web]
address = ":8080"

# 这是配置file provider以及使用的配置信息保存文件
[file]
watch = true
filename = "rules.toml"


################################################################
# Etcd configuration backend
################################################################

# Enable Etcd configuration backend
#
# Optional
#
# 这是配置provider为etcd
[etcd]

# Etcd server endpoint
#
# Required
#
# http://192.168.50.57:12379

# 配置etcd所使用的地址
endpoint = "192.168.50.57:12379"

# Enable watch Etcd changes
#
# Optional
#
watch = true

# Prefix used for KV store.
#
# Optional
#
prefix = "/traefik/dev"

# Override default configuration template. For advanced users :)
#
# Optional
#
# filename = "etcd.tmpl"

# Enable etcd TLS connection
#
# Optional
#
# [etcd.tls]
# ca = "/etc/ssl/ca.crt"
# cert = "/etc/ssl/etcd.crt"
# key = "/etc/ssl/etcd.key"
# insecureskipverify = true
```

# 这次开发的任务是给traefik添加一个个性化的provider

这次的任务是既不使用file作为provider也不是用常见的键值数据库做为provider

## 选择自制provider的原因 现有的Etcd provider存在局限

etcd provider的原理
当traefik设置了watch = true ,
traefik会向etcd订阅路径（prefix = "/traefik/dev"）的改动,现存的订阅模式是订阅prefix所制定的目录的改动
如果prefix下数据改变但是目录不改变不会触发etcd订阅数据主动发送,在测试过程中我们发现,我们向prefix目录下写入backends以及
frontends并不会修改prefix所制定的目录时间戳,因此无法到达修改局部信息主动触发的机制

其次,如果etcd服务器和traefik服务器是一对一的服务模式其实是可以做到局部数据更改prefix对应的目录修改的,现存的etcd和traefik
配合机制还存在一种原子更新机制就是在traefik etcd配置文件在watch模式下会实时订阅/traefik/alias这个key所对应的value变化
如果/traefik/alias的value存在任何修改必入value从"/traefik/v1"变味了"/traefik/v2",traefik就会使用/traefik/v2目录下的
配置做为provider,来替代上一个/traefik/v1目录下的配置信息

在这种alias模式下的话,我们可以将配置信息先写入/traefik/v2目录下,等写完/traefik/v2所有provider信息再将/traefik/v1下的
配置信息删除最后将/traefik/alias写入"/traefik/v2",一旦写入"/traefik/v2"以后由于traefik订阅了/traefik/alias数据变化
就会读取/traefik/alias下的数据"/traefik/v2"做为prefix数据provider来源

但是在公司环境下并不是一对一的服务,而是多个traefik对应一个etcd,每个traefik的prefix大致为如下结构/traefik/{cluster},
{cluster}就是traefik运行所在的集群名字一般取值为dev qas等等因此不可以简单的设置alias来更新,因为alias只可以设置一个
目录路径而不同集群下的provider数据backends以及frontends并不一样

因此公司打算开发一个定制的provider提供方式,其实跟现存的etcd模式类似,仅仅是在traefik下不再默认使用统一的traefik/alias这个
固定的alias监控key,而是将alias key的名字写入tradfik配置文件中,让不同的cluster监控不同的alias key,监控到cluster对应的
alias存在修改,traefik读取alias键值后主动向一个go的api服务请求alias键值目录下的frontends以及backends信息