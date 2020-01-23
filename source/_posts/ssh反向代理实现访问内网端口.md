---
title: ssh反向代理实现内网端口转发
copyright: true
categories: 内网渗透
tags:
  - 端口转发
  - ssh
  - 远程登陆
abbrlink: '60506580'
date: 2019-12-23 10:18:30
---

<blockquote class="blockquote-center">读书不觉已春深，一寸光阴一寸金
</blockquote>

　　为了安全起见，公司或者是学校的服务器一般只允许用户在局域网内登录，离开内网环境后就没法登录服务器，特别不方便。但是在某些情况下，我们又想和服务器进行通信的话，就需要借助端口转发来达到目的。

<!-- more -->

　　ssh是一种安全的传输协议，通常我们会用在连接服务器上比较多。不过除了这个功能以外，ssh的隧道转发功能更吸引人。

### ssh常用命令参数

![](https://img-blog.csdnimg.cn/20191223121549467.png)

1. -C 请求压缩所有数据
2. -f 告诉ssh客户端在后台运行
3. -N 告诉ssh客户端，这个连接不需要执行任何命令，仅仅做端口转发
4. -g 默认这个LocalPort端口只允许本机连接，可以通过这个参数允许别的机器连接这个端口

### ssh三种代理功能

- 正向代理（-L）：相当于 iptable 的 port forwarding
- 反向代理（-R）：相当于 frp 或者 ngrok
- socks5 代理（-D）：相当于 ss/ssr

```
ssh -C -f -N -g -L listen_port:DST_Host:DST_port user@Tunnel_Host 
ssh -C -f -N -g -R listen_port:DST_Host:DST_port user@Tunnel_Host 
ssh -C -f -N -g -D listen_port user@Tunnel_Host
```

#### **正向代理**

**-L port:host:hostport** 	 **将本地主机(客户机)的某个端口转发到远端指定机器的指定端口**

　　本地机器上分配了一个 socket 侦听 port 端口， 一旦这个端口上有了连接，该连接就经过安全通道转发出去，同时远程主机和 host 的 hostport 端口建立连接。可以在配置文件中指定端口的转发。只有 root 才能转发特权端口。IPv6 地址用另一种格式说明：port/host/hostport

- 用法1：远程端口映射到其他机器

HostB 上启动一个 PortB 端口，映射到 HostC:PortC 上，在 HostB 上运行：

```
ssh -L 0.0.0.0:PortB:HostC:PortC user@HostC
```

这时访问 HostB:PortB 相当于访问 HostC:PortC

- 用法2：本地端口通过跳板机映射到其他机器

HostA 上启动一个 PortA 端口，通过 HostB 转发到 HostC:PortC上，在 HostA 上运行：

```
ssh -L 0.0.0.0:PortA:HostC:PortC  user@HostB
```

这时访问 HostA:PortA 相当于访问 HostC:PortC

　　两种用法的区别是，第一种用法本地到跳板机 HostB 的数据是**明文**的，而第二种用法一般本地就是 HostA，访问本地的 PortA，数据被 ssh **加密传输**给 HostB 又转发给 HostC:PortC。

#### **反向代理**

**-R port:host:hostport**  	**将远程主机(服务器)的某个端口转发到本地指定机器的指定端口**

　　远程主机上分配了一个 socket 侦听 port 端口，一旦这个端口上有了连接，该连接就经过安全通道转向出去，同时本地主机和 host 的 hostport 端口建立连接。可以在配置文件中指定端口的转发。只有用 root 登录远程主机才能转发特权端口。IPv6 地址用另一种格式说明：port/host/hostport

HostA 将自己可以访问的 HostB:PortB 暴露给外网服务器 HostC:PortC，在 HostA 上运行：

```
ssh -R HostC:PortC:HostB:PortB  user@HostC
```

这时访问 HostC:PortC 就相当于访问 HostB:PortB

使用时需修改 HostC 的 /etc/ssh/sshd_config，添加：

```apacheconf
GatewayPorts yes
```

　　相当于内网穿透，比如 HostA 和 HostB 是同一个内网下的两台可以互相访问的机器，HostC是外网跳板机，HostC不能访问 HostA，但是 HostA 可以访问 HostC。

　　那么通过在内网 HostA 上运行 `ssh -R` 告诉 HostC，创建 PortC 端口监听，把该端口所有数据转发给我（HostA），我会再转发给同一个内网下的 HostB:PortB。

　　同内网下的 HostA/HostB 也可以是同一台机器，换句话说就是**内网 HostA 把自己可以访问的端口转发给了外网 HostC。**

#### **本地socks5代理**

**-D port** 	**动态端口转发**

　　本地机器上分配了一个 socket 侦听 port 端口，一旦这个端口上有了连接，该连接就经过安全通道转发出去，根据应用程序的协议可以判断出远程主机将和哪里连接。目前支持 SOCKS4 协议，将充当 SOCKS4 服务器。只有 root 才能转发特权端口。可以在配置文件中指定动态端口的转发。

　　在 HostA 的本地 1080 端口启动一个 socks5 服务，通过本地 socks5 代理的数据会通过 ssh 链接先发送给 HostB，再从 HostB 转发送给远程主机：

```
ssh -D localhost:1080 HostB
```

　　那么在 HostA 上面，浏览器配置 socks5 代理为 127.0.0.1:1080，看网页时就能把数据通过 HostB 代理出去，类似 ss/ssr 版本，只不过用 ssh 来实现。

### 虚拟机实践

![](https://img-blog.csdnimg.cn/20191223121652974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

　　这是我在虚拟机模拟的环境，假设公司内网有B机器和C机器，B机器能上外网，C机器是内网的web服务器，不能上外网，只能在内网被访问。此时我在外网想使用A机器连接到B机器，那只能反向代理，把C的端口流量转发给B，再用A去访问B。

- 启动远程服务器ssh的路由功能。在/etc/ssh/sshd_config中修改

```
GatewayPorts no 为 GatewayPorts yes
AllowAgentForwarding yes
AllowTcpForwarding yes
```

注意需要重启sshd服务systemctl sshd restart

　　如果不打开这个的话，只能在远程服务器上进行内网穿透。如果不行的话可以将被映射的端口绑定在0.0.0.0的接口上，方法是ssh加上参数-b 0.0.0.0。

- 在内网服务器上进行端口转发，并输入远程服务器的密码

```
ssh -fR 5555:127.0.0.1:22 root@192.168.30.131 -p 22 "vmstat 30"
```

　　**ssh -fR 远程服务器开放的端口:127.0.0.1:内网服务器SSH端口 root@远程服务器IP -p 远程服务器SSH端口  "vmstat 30"** 

　　-f 表示后台运行　-R表示远程端口转发　vmstat 30 是为了防止远程服务器把长时间没有通讯的链接断开。

![](https://img-blog.csdnimg.cn/20191223121924748.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

- 远程登录内网服务器

```
ssh -v -R 192.168.30.131:5555:192.168.20.129:22 root@192.168.30.130
```

![](https://img-blog.csdnimg.cn/20191223122332461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20191223122350445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)