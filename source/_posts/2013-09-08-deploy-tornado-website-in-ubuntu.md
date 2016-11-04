---
title: ubuntu下部署tornado网站
date: '2013/09/08 20:00:00'
tags:
  - ubuntu
  - tornado
---

# 1.安装Nginx

```
apt-get install nginx
```

所有的配置文件都在 /etc/nginx 下，<br>
每个虚拟主机配置文件在 /etc/nginx/sites-available 下，<br>
默认的虚拟主机的目录设置在了 /usr/share/nginx/www，<br>
启动程序文件是/usr/sbin/nginx，<br>
启动脚本 nginx 在/etc/init.d/ 下， 日志在 /var/log/nginx 中，分别是 access.log 和 error.log。

# 2.安装pip

```
apt-get install python-pip
```

# 3.安装supervisor

文档可以看 <http://supervisord.org/installing.html#installing-via-pip>

```
pip install --upgrade supervisor  
echo_supervisord_conf > /etc/supervisord.conf
```

# 4.安装virtualenv

```
pip install --upgrade virtualenv
```

# 5.安装virtualenv虚拟环境

```
virtualenv tblogenv  
source tblogenv/bin/activate
pip install tornado  
```

# 6.配置nginx

这是 /etc/nginx/sites-enabled/tblog 里的内容

```
server{
    listen 80;
    server_name dust8.com www.dust8.com blog.dust8.com;

    location ^~ /static/{
        root /usr/www/tblog;
        if ($query_string) {
            expires max;
        }
    }

    location / {
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real_IP $remote_addr;
        proxy_set_header X-Scheme $scheme;  
        proxy_pass http://127.0.0.1:8888;
    }
}  
```

# 7.配置supervisor

在 /etc/supervisord.conf 后面加入

```
[program:tblog-8888]
command=/usr/www/tblogenv/bin/python /usr/www/tblog/app.py
numproce=2
autostart=true
autorestart=true
stdout_logfile=/usr/www/tblog/python_log  
```

# 8.重启nginx和supervisor

```
/etc/init.d/nginx restart
supervisorctl restart all
```
