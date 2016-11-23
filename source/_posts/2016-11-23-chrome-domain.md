---
title: chrome有关域名的坑
date: 2016-11-23 05:15:31
tags:
  - chrome
  - 域名解析
---

本博客用的是 `github page`, 使用了自定义域名 `dust8.com`.
以前使用的是 `A` 记录, `github` 总是发送警告邮件说推荐使用 `CNAME` 记录,
所以就决定改为 `CNAME` 记录, 一改就掉坑里了.先剧透一下,所有的配置都没有问题,都是 `chrome` 惹的祸.     

经验主义有它的优点,也有很明显的缺点.打个比方你以前天天吃肉,别人就认为你今天不吃素,
虽然你今天吃肉的概率非常大,但是也不能排除你今天突然想吃素.说到这里就想起腾讯的lol游戏,
说我使用加速外挂把我封了,然而我并没有使用外挂,申诉也没人理,3天后申诉系统回给你一个空信息.
在申诉中发现它有提到使用了行为检测系统,但是点击查询却全是说系统繁忙,根本就不提供你有哪些行为.
很恼火,觉得就像它认为人都是2条腿走路,如果不是2条腿走路就不是人一样,把残疾人,狼人和猿人排除了.
如果你以前的战绩差,突然就厉害了,它就说你是挂,完全没想到是不是对手太菜，你今天超常发挥或者你提升很快.


## 检查配置    
按照 [`github` 官网的帮助文档](https://help.github.com/articles/setting-up-a-www-subdomain/)一步一步来配置,配置好后要等一段时间才有效.
等到晚上还是打不开网站,就按照帮助文档测试下    

    dig www.dust8.com +nostats +nocomments +nocmd  

    ; <<>> DiG 9.8.3-P1 <<>> www.dust8.com +nostats +nocomments +nocmd
    ;; global options: +cmd
    ;www.dust8.com.			IN	A
    www.dust8.com.		2109	IN	CNAME	dust8.github.io.
    dust8.github.io.	2110	IN	CNAME	github.map.fastly.net.
    github.map.fastly.net.	13	IN	A	151.101.16.133
    fastly.net.		6663	IN	NS	ns3.fastly.net.
    fastly.net.		6663	IN	NS	ns4.fastly.net.
    fastly.net.		6663	IN	NS	ns1.fastly.net.
    fastly.net.		6663	IN	NS	ns2.fastly.net.

可以看到已经起作用了.还是打不开网页.就清除电脑dns缓存,重启电脑,重启路由器,都没效,其实这些都不需要的,
因为上面那个命令显示dns已经解析成功了.后来还清除浏览器dns,一样无效.        


## 换浏览器   
换成 `safari` 也是一样说 `dust8.com’s server DNS address could not be found.`.
这句话很迷惑人,一直以为是 `dns` 有问题, 可以说是它的原因,也可以说不是它的原因.      
换成手机浏览器夸克居然可以打开网站,那就可以肯定是浏览器问题,因为手机和电脑用的是同一个 `wifi`.    

这里也有疑点,为什么 `safari` `chrome` 都不可以,后来才想起来 `safari` 导入了 `chrome` 的数据.
`safari` 清除数据后果然可以打开 `www.dust8.com`, 同理清除 `chrome` 数据后也可以打开了.
同时也注意到地址栏里面的网址是 `www.dust8.com` 而不是打不开时的 `dust8.com`.

## 结论   
结果已经很明了了, `chrome` 会自动记录浏览习惯来方便用户,但有时又会带来不便.
以前的域名解析是同时解析 `dust8.com` 和 `www.dust8.com` ,改了之后只解析`www.dust8.com`,
而浏览器根据以前的浏览习惯强制把 `www.dust8.com` 转到 `dust8.com`,
而 `dust8.com` 却没有设置解析, 所以才提示 `dust8.com’s server DNS address could not be found.`
