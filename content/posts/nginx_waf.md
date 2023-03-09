---
title: "nginx waf"
categories: ["waf"]
tags: 
  - waf
date: 2023-03-09T09:16:09+08:00
lastmod: 2023-03-09T09:16:09+08:00
draft: false
---

# nginx_waf介绍

nginx waf，顾名思义，就是依赖nginx的waf，也可以说是nginx插件waf。
它因为开发简便（lua的编程级灵活性）、性能卓越（nginx卓越性能）、部署简单、可扩展性高等优势，逐渐开源WAF的主流。
现在nginx官方支持[[modsecurity]]的模块，后续可以研究研究。[使用 ModSecurity/App Protect 模块构建 NGINX WAF](https://www.bilibili.com/video/BV15K41137h4/?vd_source=e986128db314de8afe0458b76b96996b)

## nginx + lua原理
![nginx_waf_20230307232048952](https://img.ivansli.com/images/nginx_waf_20230307232048952.png)
Lua可以嵌入nginx处理请求或者应答内容的过程中进行过滤。

# nginx waf安装
安装在[[CentOS7]]上（最小化安装后未设置任何内容）。

## OpenResty安装

参考[使用Nginx+Lua实现的WAF（版本v1.0）](https://github.com/unixhot/waf)

1. Yum安装OpenResty（推荐）

源码安装和Yum安装选择其一即可，默认均安装在/usr/local/openresty目录下。

```
[root@opsany ~]# wget https://openresty.org/package/centos/openresty.repo
[root@opsany ~]# sudo mv openresty.repo /etc/yum.repos.d/
[root@opsany ~]# sudo yum install -y openresty
```

2. 测试OpenResty和运行Lua

```
[root@opsany ~]# vim /usr/local/openresty/nginx/conf/nginx.conf
#在默认的server配置中增加
        location /hello {
            default_type text/html;
            content_by_lua_block {
                ngx.say("<p>hello, world</p>")
            }
        }
[root@opsany ~]# /usr/local/openresty/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/openresty-1.17.8.2/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/openresty-1.17.8.2/nginx/conf/nginx.conf test is successful
[root@opsany ~]# /usr/local/openresty/nginx/sbin/nginx
```

3. 测试访问

```
[root@opsany ~]# curl http://127.0.0.1/hello
<p>hello, world</p>
```

### WAF部署

这里是unixhot自己写好的规则，可以借鉴一下。

```
[root@opsany ~]# git clone https://github.com/unixhot/waf.git
[root@opsany ~]# cp -r ./waf/waf /usr/local/openresty/nginx/conf/
[root@opsany ~]# vim /usr/local/openresty/nginx/conf/nginx.conf
#在http{}中增加，注意路径，同时WAF日志默认存放在/tmp/日期_waf.log
#WAF
    lua_shared_dict limit 50m;
    lua_package_path "/usr/local/openresty/nginx/conf/waf/?.lua";
    init_by_lua_file "/usr/local/openresty/nginx/conf/waf/init.lua";
    access_by_lua_file "/usr/local/openresty/nginx/conf/waf/access.lua";
[root@opsany ~]# ln -s /usr/local/openresty/lualib/resty/ /usr/local/openresty/nginx/conf/waf/resty
[root@opsany ~]# /usr/local/openresty/nginx/sbin/nginx -t
[root@opsany ~]# /usr/local/openresty/nginx/sbin/nginx -s reload
```


## Nginx + Lua源码编译部署(不推荐)
，主要参考：[【技术专栏】使用开源软件建设大规模WAF集群](https://mp.weixin.qq.com/s/8bwR8CQp6uBPXx-RaqPeMw)

- 安装luajit
```shell
yum -y install gcc wget
wget http://luajit.org/download/LuaJIT-2.0.0.tar.gz
tar xf LuaJIT-2.0.0.tar.gz
cd LuaJIT-2.0.0
make
make install PREFIX=/usr/local/lj2
ln -sv /usr/local/lj2/lib/libluajit-5.1.so.2 /lib64/
```

- 安装devel_kit、lua-nginx-module
```shell
wget https://github.com/simpl/ngx_devel_kit/archive/v0.2.17rc2.zip
unzip v0.2.17rc2

wget https://github.com/chaoslawful/lua-nginx-module/archive/v0.7.4.zip
unzip v0.7.4
```

- 安装pcre、zlib
```shell
yum install -y pcre-devel zlib-devel
```

- 编译安装nginx
```shell
./configure --user=daemon --group=daemon --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_sub_module --with-http_gzip_static_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --add-module=../ngx_devel_kit-0.2.19/ --add-module=../lua-nginx-module-0.9.16/
```

- 启动并测试
```shell
[root@localhost nginx]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@localhost nginx]# /usr/local/nginx/sbin/nginx


[root@localhost nginx]# curl -I http://192.168.188.131
HTTP/1.1 200 OK
Server: nginx/1.2.9
Date: Wed, 08 Mar 2023 07:22:47 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Wed, 08 Mar 2023 06:19:50 GMT
Connection: keep-alive
Accept-Ranges: bytes
```

### WAF部署（写第一条waf拦截规则）

- 配置规则，在nginx.conf的http配置段添加如下5行
```shell
http {
    #gzip  on;

    # WAF config
    lua_need_request_body on;
    lua_package_path "/usr/local/nginx/conf/waf/?.lua";
    lua_shared_dict limit 10m;
    init_by_lua_file "/usr/local/nginx/conf/waf/init.lua";
    access_by_lua_file "/usr/local/nginx/conf/waf/waf.lua";
    
    server {
	    略略略
	略略略
```

- 增加第一条规则。
```shell
[root@localhost nginx]# mkdir -pv /usr/local/nginx/conf/waf/
[root@localhost nginx]# touch /usr/local/nginx/conf/waf/{init.lua,waf.lua}


[root@localhost nginx]# cat conf/waf/waf.lua 
if ngx.var.query_string and ngx.re.match( ngx.var.query_string, "passwd" ) then
    ngx.say("You are blocked by douge!");
    return ngx.exit(ngx.HTTP_FORBIDDEN);
end
```

- 启动并测试
```shell
[root@localhost nginx]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@localhost nginx]# /usr/local/nginx/sbin/nginx


[root@localhost nginx]# curl -I http://192.168.188.131
HTTP/1.1 200 OK
Server: nginx/1.2.9
Date: Wed, 08 Mar 2023 07:22:47 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Wed, 08 Mar 2023 06:19:50 GMT
Connection: keep-alive
Accept-Ranges: bytes


[root@localhost nginx]# curl http://192.168.188.131/?file=/etc/passwd
You are blocked by douge!
```

# Nginx+Lua编写WAF
[Nginx+Lua编写WAF](Nginx+Lua编写WAF.md)

# waf集群搭建

- 服务器角色
	- 负载均衡集群
		- 需要使用反向代理模式，后端web服务器不允许暴露出来。
	- WAF服务集群
	- WAF日志处理集群
	- WAF配置管理集群
	- web服务集群
- 以上服务器角色都支持横向扩展，几乎没有性能瓶颈。
![WAF_20230308144505681](https://img.ivansli.com/images/WAF_20230308144505681.png)

## 配置管理

在大规模的WAF集群中，不太可能手动去更新规则。第一服务器数量太多，效率慢；第二，只要是人工还极易出错。
那么就需要一个集群专门管理WAF规则，然后其他WAF服务器去同步这个规则。常用的配置同步方式，使用ZooKeeper。

### ZooKeeper实现nginx配置订阅

在最简化设计时，我们可以让每台WAF的配置完全一样，这样的好处是可以随时添加服务器，任何一个服务器宕机都不会影响WAF服务，而且整个系统的架构会非常简洁。每台WAF只要使用python脚本订阅ZK中的配置，一旦发生变更，获取最新配置，更新相关配置文件，给nginx发送信号热加载新配置文件即可（nginx -s reload）。

![nginx_waf_20230308161433221](https://img.ivansli.com/images/nginx_waf_20230308161433221.png)

### ZooKeeper安装

- **安装jdk**
```shell
yum -y install java-11-openjdk
```

- **下载并安装zk**
```shell
wget https://dlcdn.apache.org/zookeeper/zookeeper-3.7.1/apache-zookeeper-3.7.1-bin.tar.gz
tar xf apache-zookeeper-3.7.1-bin.tar.gz
cd apache-zookeeper-3.7.1-bin/conf
cp zoo_sample.cfg zoo.cfg

cd ../bin/
sh zkServer.sh start
```

- **zk典型配置**
```shell
[root@localhost apache-zookeeper-3.7.1-bin]# grep -v "^#" conf/zoo.cfg 
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/var/lib/zookeeper
clientPort=2181

server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```
建议至少三个zk实例，而且分布在不同服务器上，可以和WAF混合部署，因为对性能消耗很小。

### 订阅WAF规则变更

核心python代码为：
```python
#创建zookeeper节点 
>>> zookeeper.create(zk,"/zk_waf_rule")

#获取节点信息 
>>> zookeeper.get(zk,"/zk_waf_rule")

#定义一个监听器,订阅配置 
>>> def myWatch(zk, type, state,path): 
...     print "zk:",str(type),str(state),str(path) 
...     zookeeper.get(zk,path,myWatch) 

>>> data=zookeeper.get(zk,"/zk_waf_rule",myWatch)
```

## 日志处理

![nginx_waf_20230308170047364](https://img.ivansli.com/images/nginx_waf_20230308170047364.png)

日志处理也基于流行的storm做实时流式处理，基于spark做离线分析，基于hdfs做离线存储，如果有实时检索原始日志的需求，可以引入es。细节可以参考之前的文章[[搭建开源SIEM平台]]。

这部分最基础的功能是：（实时和离线处理）
- 统计实时拦截报表
- 统计实时访问PV/UV
- 离线分析小时/天/周/月的PV/UV
- 离线分析小时/天/周/月的拦截报表
- 离线保存原始日志

加分功能是：
- 跑HMM分析漏报
- 跑语义分析漏报

## WAF集群总结

生产环境中，远比这个复杂，自研需要谨慎，BAT3以及传统4强的WAF团队人数据说都不是个位数：
- nginx层面需要大量优化提高性能
- WAF规则需要跟进最新的漏洞，运维量大
- 开源大数据框架storm、hdfs、spark稳定的运行绝非易事，需要专人维护
- nginx+lua架构出于性能考虑单向匹配规则，误报漏报很难取舍

## WAF集群监控

**平时多监控，出事少背锅**

- 短时间大量拦截的监控
	- 应对场景：新上线的规则或者是业务新上线某个功能的时候，可能跟我们的规则有一定的冲突或者误拦。
	- 需要判断是因为新的业务、还是新的功能，还是真实的攻击
- 报文处理延时的监控
	- 上了WAF之后，可能会造成业务响应的延时会变大。
	- 我们要监控，到底是后端的业务造成的延时大，还是waf导致的延时大。
- 配置一致性的监控
	- 配置不一致会导致偶尔行，偶尔不行的情况。
	- 以及要监控WAF配置是否生效。

