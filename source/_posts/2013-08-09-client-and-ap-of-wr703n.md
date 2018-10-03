---
title: 用wr703n同时做client和ap
date: '2013/08/09 20:00:00'
tags:
  - wr703n
  - ap
---

我这个TL-WR703N买了好久，都没发挥它的作用。 刚好有个这样的需求：路由器能同时接收和发射无线信号。就把它刷了个openwrt。

用下面的文件来讲解下。

"aaa"是我想要接收的信号，它绑定了物理地址，一般手机都改不了物理地址。<br>
"hi"是我要发射的信号，设了密码。

所有的设置好后记得用 **wifi** 或者 **reboot** 命令重启设置。

# 这是`/etc/config/wireless` 文件

```
config wifi-device 'radio0'
        option type 'mac80211'
        option channel '11'
        option macaddr 'ec:88:8f:dc:8f:88'
        option hwmode '11ng'
        option htmode 'HT20'
        list ht_capab 'SHORT-GI-20'
        list ht_capab 'SHORT-GI-40'
        list ht_capab 'RX-STBC1'
        list ht_capab 'DSSS_CCK-40'

config wifi-iface
        # 设置设备为上面的‘radio0’
        option device 'radio0'
        # 设置网络为无线局域网
        option network 'lan'
        # 设置路由器模式为‘ap’模式
        option mode 'ap'
        # 设置无线网络名为‘hi’
        option ssid 'hi'
        # 设置无线网络的加密方式
        option encryption 'psk2'
        # 设置无线网络的密码
        option key 'aaaaaaaa'

config wifi-iface
        option device 'radio0'
        option network 'wan'
        option mode 'sta'
        option ssid 'aaa'
        option encryption 'psk2'
        option key 'qqqqqqqq'
        # 设置自定义的网卡物理地址，一定要大写
        option macaddr '00:20:03:90:7C:28'
```

# 这是 `/etc/config/network` 文件

```
config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config interface 'lan'
        option ifname 'eth0'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'

config interface 'wan'
        option ifname 'wlan0'
        option proto 'dhcp'
```

# 这是 `/etc/config/dhcp` 文件

```
config dnsmasq
        option domainneeded     1
        option boguspriv        1
        option filterwin2k      0  # enable for dial on demand
        option localise_queries 1
        option rebind_protection 1  # disable if upstream must serve RFC1918 addresses
        option rebind_localhost 1  # enable for RBL checking and similar services
        #list rebind_domain example.lan  # whitelist RFC1918 responses for domains
        option local    '/lan/'
        option domain   'lan'
        option expandhosts      1
        option nonegcache       0
        option authoritative    1
        option readethers       1
        option leasefile        '/tmp/dhcp.leases'
        option resolvfile       '/tmp/resolv.conf.auto'
        #list server            '/mycompany.local/1.2.3.4'
        #option nonwildcard     1
        #list interface         br-lan
        #list notinterface      lo
        #list bogusnxdomain     '64.94.110.11'

config dhcp lan
        option interface        lan
        option start    100
        option limit    150
        option leasetime        12h

config dhcp wan
        option interface        wan
        # 注释掉下面这行
        # option ignore  1
```

# 参考内容：

1.[Openwrt Wan桥接Lan](http://see.sl088.com/wiki/Openwrt_Wan%E6%A1%A5%E6%8E%A5Lan)<br>
2.[OpenWrt/Tplink WR-703N - TUNA/CA wiki](http://wiki.tuna.tsinghua.edu.cn/OpenWrt/Tplink%20WR-703N)
