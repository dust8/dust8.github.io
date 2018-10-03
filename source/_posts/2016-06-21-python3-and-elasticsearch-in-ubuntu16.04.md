---
title: python3和elasticsearch在ubuntu16.04
date: 2016-06-21 14:46:05
tags:
 - elasticsearch
 - ik
---

今天搞了个新 `vps` , 用的是 `ubuntu 16.04`, 默认是 `python3` .     

# python
 - 默认是 `python3`, 输入 `python` 无效.   
 - 用 `apt install python3-pip` 安装 `pip3`, 命令是 `pip3` .    
 - 安装 `bloomfilter` 出现 `'x86_64-linux-gnu-gcc' failed` ,
 用 `apt install gcc python3-dev` 来解决.    


# elasticsearch    
## 安装elasticsearch      

      wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

      echo "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list

      sudo apt-get update && sudo apt-get install elasticsearch

## 启动elasticsearch

      service elasticsearch restart

## 查看配置

      ps -ef | grep elasticsearch

可以看到各种目录    

## 安装elasticsearch-analysis-ik    
因为 `elasticsearch` 是 `2.3.3` , 所以 `IK version` 要 `1.9.3`.    

    cd /tmp
    curl https://codeload.github.com/medcl/elasticsearch-analysis-ik/zip/v1.9.3 -o m.zip
    apt install unzip
    unzip m.zip
    cd elasticsearch-analysis-ik-1.9.3

因还没有安装 `maven`, 不能使用 `mvn`.    

    apt install maven
    mvn package

居然还报错 `Assembly: null is not configured correctly: Assembly ID must be present and non-empty.`,    

    sed -i '/<assembly>/a\ <id>releases</id>' src/main/assemblies/plugin.xml
    mvn package
    unzip /tmp/elasticsearch-analysis-ik-1.9.3/target/releases/elasticsearch-analysis-ik-1.9.3-releases.zip -d /usr/share/elasticsearch/plugins/ik
    service elasticsearch restart
