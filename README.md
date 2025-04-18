# BGPserver
a live BGP server peer FOR any country  实时获取全球BGP 路由软件<br>
路由采集信息以中国地区为优化any cast类型的路由
###    编辑config.ini 配置文件 设定相关参数
###    运行命令 参数 -c 为指定config.ini 文件路径（相对路径即可）
###    -l 参数 指定需要获得的国家路由 例如CN、US 特定两个参数NCN 所有非CN路由 ALL 为全球BGP路由 
. 例如 bgp -c ./config.ini -l ALL  为全球BGP bgp -c ./config.ini -l US 为美国路由 无需区分大小写
. bgp -c ./config.ini -l NCN 为非CN路由<br>
. config.ini配置文件只需简单设置 服务器routerid peer nexthop 和peer端IP 和peer 端ASN 和updateSource 即可 <br>
##### 程序运行需要两个文件 map.json.gz、chinaBGPZip.gob 务必放置于bgp文件同一目录下 
##### 建议在能科学上网的环境下运行 否则不一定保证能接收到实时路由
##### BGP 通信端口为TCP 179 请注意开发防火墙
##### *仅用于学习目的*

Dockerfile
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# 先克隆源码，编译好可运行文件，再copy运行需要的文件到app目录
# 可带人参数 TYPE，默认为‘CN’
# 最简linux环境下运行
# 支持默认值（${VAR:-default} 语法）

FROM golang:alpine AS builder

RUN apk add --no-cache git
RUN git clone --depth 1 https://github.com/TaylorGotIT/BGPserver.git && \
    cd BGPserver/BGP && \
    CGO_ENABLED=0 go build -o bgp bgp.go

FROM alpine:latest

COPY --from=builder /go/BGPserver/BGP/bgp /app/
COPY --from=builder /go/BGPserver/BGP/config.ini /app/
COPY --from=builder /go/BGPserver/BGP/map.json.gz /app/
COPY --from=builder /go/BGPserver/BGP/ChinaBGPZip.gob /app/

WORKDIR /app

ENTRYPOINT ["/bin/sh", "-c", "exec ./bgp -c config.ini -l \"${ROUTE_TYPE:-CN}\""]
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

构建Docker镜像
sudo docker build -t bgpserver .

$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
bgpserver    latest    f909e1837bf7   About a minute ago   38.4MB

上传镜像到Docker Hub
登录Docker
$ sudo docker login
[sudo] password for taylorg:
USING WEB-BASED LOGIN
To sign in with credentials on the command line, use 'docker login -u <username>'
Your one-time device confirmation code is: BQNW-VCVP
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate
Waiting for authentication in the browser…

标记名称
$ sudo docker tag bgpserver taylorgogo/bgpserver

上传到Docker Hub
$ sudo docker push taylorgogo/bgpserver
Using default tag: latest
The push refers to repository [docker.io/taylorgogo/bgpserver]
5f70bf18a086: Mounted from library/golang
a08fbab88915: Pushed
b20467339e3c: Pushed
1985f2b98f55: Pushed
2ed4c309a65c: Pushed
08000c18d16d: Mounted from library/golang
latest: digest: sha256:db7641a80d4d13a96b66e189a3bc2f8fa3862a5594f9d62b1ba055114db3e13b size: 1575

MyHub
https://hub.docker.com/repositories/taylorgogo


Vyos使用构建的镜像

添加镜像
$ add container image  taylorgogo/bgpserver

查看镜像
$ show container image
REPOSITORY                                    TAG      IMAGE ID      CREATED         SIZE
docker.io/taylorgogo/bgpserver                latest   f909e1837bf7  29 minutes ago  38.7 MB

