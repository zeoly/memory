# Rustdesk 搭建

## 背景

对于有远程控制的需求，但todesk收费，向日葵又慢的情况下，可以搭建自己的远程服务，实现点对点传输控制，关键是搭建简单而且免费

## 服务搭建

由于已经有一台unraid系统的服务器，以及有公网ip的前提，决定使用docker搭建服务

### 拉取镜像

拉取镜像时，可以自行查找可用源

```bash
sudo docker image pull rustdesk/rustdesk-server
```

### 启hbbs

注意端口（udp），映射的路径（会存放证书），最后的命令

```bash
sudo docker run --name hbbs -p 21115:21115 -p 21116:21116 -p 21116:21116/udp -p 21118:21118 -v /mnt/user/appdata/rustdesk:/root -td rustdesk/rustdesk-server hbbs -r [域名或公网IP]
```

启动完成后，可从日志中看到

```
[2024-10-29 04:18:12.568615 +00:00] INFO [src/rendezvous_server.rs:1205] Key: [一串字符]
```

### 启hbbr

同样注意端口，以及最后的命令。此启动无须指定域名

```bash
sudo docker run --name hbbr -p 21117:21117 -p 21119:21119 -v /mnt/user/appdata/rustdesk:/root -td rustdesk/rustdesk-server hbbr
```

### 其他

开放端口

默认情况下，hbbs 监听21115(tcp), 21116(tcp/udp), 21118(tcp)，hbbr 监听21117(tcp), 21119(tcp)。务必在防火墙开启这几个端口， 请注意21116同时要开启TCP和UDP 。其中21115是hbbs用作NAT类型测试，21116/UDP是hbbs用作ID注册与心跳服务，21116/TCP是hbbs用作TCP打洞与连接服务，21117是hbbr用作中继服务, 21118和21119是为了支持网页客户端。如果您不需要网页客户端（21118，21119）支持，对应端口可以不开。

TCP( 21115, 21116, 21117, 21118, 21119 )
UDP( 21116 )

## 客户端配置

在官网（目前跳转github）下载客户端，安装后进入 **设置** -> **网络**，配置如下：

- ID服务器：域名:21116
- 中继服务器：域名:21117
- Key: hbbs服务日志中的key值

回到主页，左下角显示就绪，表示连接成功

