#PlutoX

### A proxy tool for macOS.

<a href="https://itunes.apple.com/cn/app/plutox/id1176077430?mt=12"><img src="https://www.plutox.top/img/download.png" width = "210" height = "80" alt=align=center /></a>


## 自定义规则
[ss(r)内置规则模板](https://github.com/nidom/PlutoX/blob/master/config.ss.general.yaml)

##语法
自定义配置使用YAML标准格式，通过 [YAML官网](http://www.yaml.org/) 或 [docs.ansible.com](http://docs.ansible.com/ansible/YAMLSyntax.html) 了解语法。

> Note：YAML对缩进非常严格，并且只能用空格。


##基本概念
rule 和 adapter 是自定义配置的两个核心概念：rule表示规则，用于匹配网络请求，网络请求匹配到某条规则之后，会被分配到相应的adapter上。示例是一条 Country rule，当请求对应的IP归属中国，那么请求将被分配到名为 adapterName 的 adapter 上。

```
 - type: country
   country: CN
   match: true
   adapter: adapterName
```
       
一个自定义规则中可以包含多个 rule 和 adapter。除了 direct 这个内置的 adapter，每个 adapter 需要定义唯一的 id 。每条 rule 我们需要给它指定相应的 adapter 。

一个自定义配置的框架如下：

```

adapter:
 - id: adapter1
   ...
 - id: adapter2
   ...
rule:
 - type: iplist
   adapter: adapter1
   ...
 - type: country
   adapter: adapter2
   ...
```

##Adapter详解
PlutoX支持以下几个类型的Adapter
####HTTP(S)
HTTP代理

```
 - id: httpAdapterName
   type: HTTP
   host: http.host
   port: 8080
   auth: true #可选，是否需要身份验证
   username: proxy_username #可选
   password: proxy_password #可选
```
HTTPS代理

```
 - id: httpAdapterName
   type: SHTTP
   host: http.proxy.connect.via.https
   port: 8080
   auth: true #可选，是否需要身份验证
   username: proxy_username #可选
   password: proxy_password #可选
```

####SOCK5
socks5代理

```
 - id: socks5AdapterName
   type: socks5
   host: socks5.host
   port: 3128
```
####SS
SS代理

```
 - id: ssAdapterName
   type: ss
   host: ss.host
   port: 1024
   method: AES-128-CFB   #可选   AES-128-CFB, AES-192-CFB, AES-256-CFB, chacha20, salsa20, rc4-md5
   password: ss_password
   protocol: origin   #可选 origin(无)、verify_sha1(OTA)
   obfs: origin      #可选 origin(无)、http_simple、tls1.2_ticket_auth
   obfs_param: ""
```
####Speed
speed用于选择最快连接成功的线路，
>每次网络请求， speed 都会连接所有线路，选择最快连接成功的线路。

```
 - id: speedAdapterName
   type: speed
   adapters:
    - id: adapter1
      delay: 300 # 延时300毫秒
    - id: adapter2
      delay: 300
    - id: adapter3
      delay: 300
    - id: direct
      delay: 0
```
####Reject
reject会抛弃网络请求

```
 - id: reject
   type: reject
   delay: 300 #延时300毫秒
```
####Direct
直连网络

```
内置 adapter ，不需要定义
```
##Rule详解
>rule 自上而下匹配的，以第一次匹配为准，所以rule书写的前后顺序会影响结果

####Country
同个IP判断归属国家

```
 - type: country
   country: CN # ISO Country code,‘--’表示未知国家
   match: true #是否匹配
   adapter: adapterName
```
####Domain List
匹配域名列表，criteria中的p,k,s,r分别表示 prefix (前缀)，keyword(关键词),suffix(后缀),regex(正则表达式)


```
 - type: domainlist
   criteria:
    - p,ad
    - k,google
    - s,ad.com
    - r,google+.\.com
   adapter: adapterName
```

####IP List
匹配IP列表

```
 - type: iplist
   criteria:
    - 127.0.0.0/8
    - 192.168.0.0/16
    - 10.0.0.0/8
    - 224.0.0.0/8
    - 169.254.0.0/16
   adapter: adapterName
```
####DNSFail
DNS解析错误的时候匹配

```
 - type: dnsfail
   adapter: adapterName
```
####All
匹配所有网络请求

```
 - type: all
   adapter: adapterName
```
##示例
####所有请求走SS代理
这是最简单的一个自定义规则

```
adapter:
 - id: ss_proxy
   type: ss
   method: ss_method
   host: ss_host
   port: ss_port
   password: ss_password
rule:
 - type: all
   adapter: ss_proxy
```
####稍微复杂一点的规则
内网IP和中国IP不走代理，美国和日本的网站连接位于东京的SS代理，其他国外网站连接香港的HTTPS代理。
>注：不同的 rule 可以指向相同的 adapter


```

adapter:
 - id: tokyo_ss_proxy
   type: ss
   method: ss_method
   host: ss_host
   port: ss_port
   password: ss_password
 - id: hk_https_proxy
   type: http
   host: http.host
   port: 8080
   secured: true
   auth: true
   username: proxy_username
   password: proxy_password
rule:
 - type: domainlist
   criteria:
    - 127.0.0.0/8
    - 192.168.0.0/16
    - 10.0.0.0/8
    - 224.0.0.0/8
    - 169.254.0.0/16
   adapter: direct
 - type: country
   country: US
   match: true
   adapter: tokyo_ss_proxy
 - type: country
   country: JP
   match: true
   adapter: tokyo_ss_proxy
 - type: country
   country: CN
   match: true
   adapter: direct
 - type: all
   adapter: hk_https_proxy
```
