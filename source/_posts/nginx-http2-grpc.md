---
title: nginx-http2-grpc
date: 2018-03-23 16:43:36
tags:
    - nginx
    - http2
    - grpc
---

<!--
  ~ Copyright 2022 kwanhur
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~ http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  ~
-->

NGINX 官方已发布[1.13.10](http://nginx.org/en/CHANGES)版本，其最主要更新功能点莫过于支持 gRPC 服务代理，即新增的模块[ngx_http_grpc_module](http://nginx.org/en/docs/http/ngx_http_grpc_module.html)。

这意味着可以很容易复用 Nginx 现有基础设施，简单几步即可构建出高可用的 gRPC 服务端集群，具体构建概要如下：

<!--more-->

> 搭建 go-grpc 服务，参见 [https://grpc.io/docs/quickstart/go.html](https://grpc.io/docs/quickstart/go.html)

```bash
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld

#启动服务端，默认端口 50051
$ go run greeter_server/main.go

#测试客户端
$ go run greeter_client/main.go

#交互成功可见输出结果
Greeting: Hello world
```

> 搭建 nginx-grpc 代理
>> nginx 构建支持 http2+grpc

```bash
$ wget http://nginx.org/download/nginx-1.13.10.tar.gz -O /tmp/nginx-1.13.10.tar.gz

$ cd /tmp && tar xzf nginx-1.13.10.tar.gz && cd nginx-1.13.10

$ ./configure --with-http_ssl_module --with-http_v2_module && make -j4 && make install && /usr/local/nginx/sbin/nginx -V
```

>> 配置 nginx 支持 grpc 代理

```
#vhosts/grpc.conf 配置 demo 文件
upstream grpc.server{
    server 127.0.0.1:50051;
}

server {
    listen 80 http2;
    access_log grpc.access.log main;
        
    location / {
        grpc_pass grpc.server;
    }
}
```

>> 从上述简单的配置可看出，grpc 模块可以和 upstream 模块结合使用，这意味着可复用 upstream 模块[高可用配置](http://nginx.org/en/docs/http/ngx_http_upstream_module.html),
当然，grpc 模块也提供类似配置，如[grpc_next_upstream](http://nginx.org/en/docs/http/ngx_http_grpc_module.html#grpc_next_upstream)等。
>>
> 验证 nginx-grpc 代理
>> 启动 nginx 代理服务

```bash
$ sudo /usr/local/nginx/sbin/nginx -t && sudo /usr/local/nginx/sbin/nginx
```

> grpc 服务测验

```bash
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld

#！注意修改客户端连接服务端口为 80
$ go run greeter_client/main.go

#交互成功可见输出结果
Greeting: Hello world
#nginx 访问日志可见输出
127.0.0.1 - - [23/Mar/2018:17:21:10 +0800] "POST /helloworld.Greeter/SayHello HTTP/2.0" 200 18 "-" "grpc-go/1.11.0-dev" "-"
```

总体可见，通过 nginx 可以很轻松的构建出高可用的 grpc 服务代理。
