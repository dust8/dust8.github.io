---
title: sqlserver与clr程序集
date: 2018-08-21 05:15:31
tags:
    - dba
    - sqlserver
    - clr
---

最近碰过一个需求，要把 `sqlserver` 里面的几个字段拼接并做 `md5` 处理，在插入为一个 `guid` 字段。由于数据库安装的是中文，用数据库的 `HASHBYTES` 做 `md5` 碰到中文会和正常的不一致，因为它是用 `gb2312` 编码的，正常的应该是用 `utf8` 编码。后来的网上找到可以用 `SQL CLR 用户定义的函数` 来实现。

## 新建项目
![新建项目](/assert/2018-08-21-1.png)    

## 添加新建项
![添加新建项](/assert/2018-08-21-2.png)    

## 实现函数
```cshare
using System;
using System.Data;
using System.Data.SqlClient;
using System.Data.SqlTypes;
using Microsoft.SqlServer.Server;
using System.Security.Cryptography;
using System.Text;

public partial class UserDefinedFunctions
{
    [Microsoft.SqlServer.Server.SqlFunction]

    /// <summary>
    /// md5加密
    /// </summary>
    /// <param name="text"></param>
    public static SqlString MD5Encrypt(SqlString text)
    {
        string hash = string.Empty;
        MD5CryptoServiceProvider md5provider = new MD5CryptoServiceProvider();
        byte[] bytes = md5provider.ComputeHash(new UTF8Encoding().GetBytes(text.ToString()));
        for (int i = 0; i < bytes.Length; i++)
        {
            hash += bytes[i].ToString("x2");
        }
        return new SqlString(hash.ToString());
    }

    /// <summary>
    /// 字符串转guid
    /// </summary>
    /// <param name="text"></param>
    /// <returns></returns>
    public static SqlString Guid(SqlString text)
    {
        Guid guid = new Guid(text.ToString());
        return new SqlString(guid.ToString());
    }

}
```

## 生成dll


## 注册dll
![注册dll](/assert/2018-08-21-3.png)   

## 使用
```sql
USE temp;
GO

-- 注册自定义函数
CREATE FUNCTION dbo.MD5Encrypt(@src NVARCHAR(4000))
RETURNS NVARCHAR(32)
EXTERNAL NAME SqlCLR.UserDefinedFunctions.MD5Encrypt;
GO

CREATE FUNCTION dbo.Guid(@src NVARCHAR(32))
RETURNS NVARCHAR(50)
external name SqlCLR.UserDefinedFunctions.Guid;

-- 开启clr 支持
EXEC sp_configure 'clr enabled';  
EXEC sp_configure 'clr enabled' , '1';  
RECONFIGURE;    

 
select dbo.MD5Encrypt('我'),dbo.Guid(dbo.MD5Encrypt('我'))
```
![注](/assert/2018-08-21-4.png)   


备注：
- [HASHBYTES](https://docs.microsoft.com/zh-cn/sql/t-sql/functions/hashbytes-transact-sql?view=sql-server-2017)
- [clr enabled 服务器配置选项](https://docs.microsoft.com/zh-cn/sql/database-engine/configure-windows/clr-enabled-server-configuration-option?view=sql-server-2017)
- [用户定义函数](https://docs.microsoft.com/zh-cn/sql/2014/relational-databases/user-defined-functions/user-defined-functions?view=sql-server-2017)