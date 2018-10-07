---
title: powershell查询dns
date: 2018-10-07 05:15:31
tags:
    - powershell
    - dns
---

最近碰到把一个二级域名误解析到了2个ip上,导致接口调不通.
应为dns是可以这样操作的,所以添加的时候没有报错.因此需要添加域名指向后要确认下是否指向正确.


## 示例
### query_dns.ps1
```powershell
function PrintTitle($title){
    $sep = '+' *80
    $sep
    '+        {0}{1}+' -f $title,(' '*(80-10-$title.length))
    $sep
}

# 获取本ds 服务器地址
PrintTitle('1. DnsClientServerAddress')
Get-DnsClientServerAddress -AddressFamily IPv4
''

# 清空 dns 缓存 
PrintTitle('2. Flushdns')
ipconfig /flushdns
''

# 查询域名的 dns
PrintTitle('3. Resolve-DnsName for domain_waiting_to_be_queried.txt')
Get-Content 'domain_waiting_to_be_queried.txt' | Resolve-DnsName -Type A | Format-Table -Autosize
```
### domain_waiting_to_be_queried.txt
```
google.com
bing.com
baidu.com
```


## 输出
```
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+        1. DnsClientServerAddress                                             +
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

InterfaceAlias               Interface Address ServerAddresses
                             Index     Family
--------------               --------- ------- ---------------
以太网                              21 IPv4    {}
vEthernet (DockerNAT)               49 IPv4    {}
本地连接* 11                        14 IPv4    {}
本地连接* 1                          4 IPv4    {}
以太网 2                            16 IPv4    {}
WLAN                                15 IPv4    {192.168.0.1}
蓝牙网络连接                        12 IPv4    {}
Loopback Pseudo-Interface 1          1 IPv4    {}
vEthernet (Default Switch)          24 IPv4    {}

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+        2. Flushdns                                                           +
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Windows IP 配置

已成功刷新 DNS 解析缓存。

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+        3. Resolve-DnsName for domain_waiting_to_be_queried.txt               +
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



Name       Type TTL Section IPAddress
----       ---- --- ------- ---------
google.com A    300 Answer  216.58.220.206
bing.com   A    300 Answer  204.79.197.200
bing.com   A    300 Answer  13.107.21.200
baidu.com  A    386 Answer  123.125.115.110
baidu.com  A    386 Answer  220.181.57.216
```


## 引用
- [PowerShell如何解析DNS](https://www.pstips.net/question/3966.html)
- [DNS Client Cmdlets in Windows PowerShell](https://msdn.microsoft.com/zh-cn/library/windows/desktop/jj590772(v=wps.630).aspx)