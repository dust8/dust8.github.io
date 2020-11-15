---
title: 使用pt-archiver数据库归档
date: 2020-11-15 05:15:31
tags:
  - pt-archiver
  - mysql
---

## 安装 pt-archiver
`percona-toolkit` 只支持 `linux`.
```bash
# 下载安装包
wget https://www.percona.com/downloads/percona-toolkit/3.2.1/binary/debian/bionic/x86_64/percona-toolkit_3.2.1-1.bionic_amd64.deb

# 安装
sudo dpkg -i percona-toolkit_3.2.1-1.bionic_amd64.deb

# 如果使用 --ask-pass 需要安装libterm-readkey-perl
sudo apt install libterm-readkey-perl
```

## 归档策略
先迁移数据到归档表, 在删除原本的已迁移数据

### 迁移全部数据,并替换为更新了的数据
`--where '1=1'` 表示迁移全表. `--no-delete` 表示不删除原表. `--replace`, 有些是关联的外键,必须有, 而且数据又会更新, 所以使用替换. `--nosafe-auto-increment` 表示迁移条件的所以数据, 不然会保留一条数据, 用来避免 `mysql` 重启后没有记住最后记录的自增 `id`, 要慎用.  `--ask-pass` 表示输入密码

```bash
pt-archiver --source h=192.168.0.122,P=3306,u=root,D=oxbfund,t=users_userprofile,A=utf8 --dest h=192.168.0.122,P=3306,u=root,D=axbfund,t=users_userprofile,A=utf8 --charset=utf8 --where '1=1' --progress 500 --txn-size=1000 --statistics --no-delete --replace  --nosafe-auto-increment --ask-pass
```

### 迁移需要归档的老数据
`--no-check-charset` 是不检查数据库的编码
```bash
pt-archiver --source h=192.168.0.122,P=3306,u=root,D=oxbfund,t=trade_payrecorder,A=utf8 --dest h=192.168.0.122,P=3306,u=root,D=axbfund,t=trade_payrecorder,A=utf8 --charset=utf8 --where 'classorder_info_id in (SELECT order_info_id FROM trade_orderresell where resell_status="RESELL_FINISHED" and match_time < "2020-07-01")' --progress 500 --txn-size=1000 --statistics --no-delete  --no-check-charset --ask-pass
```

### 删除已经迁移的数据
`--purge` 表示清除
```bash
pt-archiver --source h=192.168.0.122,P=3306,u=root,D=oxbfund,t=trade_payrecorder,A=utf8 --charset=utf8 --where 'classorder_info_id in (SELECT order_info_id FROM trade_orderresell where resell_status="RESELL_FINISHED" and match_time < "2020-07-01")' --progress 500 --txn-size=1000 --statistics --no-check-charset  --purge --ask-pass
```

## 参考链接
- [Percona Toolkit Documentation](https://www.percona.com/doc/percona-toolkit/LATEST/index.html)
- [Percona-Toolkit 之 pt-archiver 总结](https://www.cnblogs.com/dbabd/p/10721857.html)
- [优雅地使用pt-archiver进行数据归档](https://juejin.im/post/6844903517954506759)
- [MySQL 大表处理](https://yangxikun.github.io/mysql/2019/10/29/mysql-big-table.html)
- [利用 pt-archiver 归档 关联表](https://www.jianshu.com/p/24a898fa62c0)