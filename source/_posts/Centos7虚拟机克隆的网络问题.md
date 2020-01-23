---
title: Centos7虚拟机克隆的网络问题
copyright: true
categories: 问题解决
tags:
  - 网卡
  - Centos7
  - 克隆
abbrlink: 2e7f84f8
date: 2020-01-07 11:54:38
---

<blockquote class="blockquote-center">壮心未与年俱老，死去犹能作鬼雄
</blockquote>

　　今天克隆了centos7虚拟机，打开却发现一直没有网卡信息，后来才想起克隆会造成网卡信息的冲突，得手动进行修改才能获取ip，所以记录一下避免以后重走老路浪费时间。

<!-- more -->

### 遇到的问题

打开克隆好的虚拟机，查看网卡信息发现没有IP，对了，我这里的虚拟机是双网卡

![](https://img-blog.csdnimg.cn/20200107120022462.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 解决方法

1、进入`/etc/udev/rules.d/`这个目录，删除文件`70-persistent-ipoib.rules`

```
[root@Centos ~]#  cd /etc/udev/rules.d/
[root@Centos rules.d]#  rm -f 70-persistent-ipoib.rules
```

2、修改网卡配置文件`/etc/sysconfig/network-scripts/ifcfg-ens33`，这里具体看大家的网卡名字，我的是ens33

- 删除UUID这一行，因为每张网卡的mac地址是不一样的，所以UUID也是不一样的

- 修改HWADDR为虚拟机克隆后的MAC地址

![](https://img-blog.csdnimg.cn/20200107115113530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)


　　ip addr可以看到mac地址，但是还没修改网卡配置信息之前，这里是不能看到具体的ens33、37显示，只能看到类似于下方显示的virbr0名字

3、然后如法炮制，copy一份修改好的ens33配置文件，修改为另外一块网卡正确的mac地址

```
[root@Centos network-scripts]# cp ifcfg-ens33 ifcfg-ens37
```


我这里看到的名字是ens37，所以我得复制为ens37，然后再修改相应的mac和name

![](https://img-blog.csdnimg.cn/20200107115223711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

4、最后重启一下网络服务，再检查一下IP信息就ok了

```
[root@Centos ~]#  systemctl restart network.service
```

![](https://img-blog.csdnimg.cn/2020010711485876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 注意

如果启动网络服务时出现Device eno16884287 does not seem to be present错误

- 修改网卡配置文件/etc/sysconfig/network-scripts/ifcfg-eno16884287, 修改Device和Name的名称， 如修改为“eth33”；
- 确认网卡配置文件中“HWADDR”虚拟机的MAC地址是否正确；
- 将网卡配置文件名重命名为/etc/sysconfig/network-scripts/ifcfg-eth33。

```
 [root@Centos network-scripts]#  mv ifcfg-eno16884287 ifcfg-eth33
```


如果启动网络服务时出现Error, some other host already use address错误

- 出现该错误说明同一个网段中有主机已经占用该虚拟主机配置的IP地址， 需要重新配置一个尚未使用的IP地址。