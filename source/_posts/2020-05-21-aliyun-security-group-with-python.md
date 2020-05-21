---
title: 使用 python 来自动修改阿里云安全组
date: 2020-05-21 05:15:31
tags:
  - python
---

有台服务器只给内部人员用, 所以就添加了白名单. 但是 ip 总是变, 手动改太麻烦了,
就写了个定时查询出 ip, 发现 ip 变了就自动修改阿里云的安全组设置的脚本.

```python
#!/usr/bin/env python
# coding=utf-8
# https://helpcdn.aliyun.com/document_detail/25699.html
# https://helpcdn.aliyun.com/document_detail/25554.html
import logging
import time

import requests
from aliyunsdkcore.acs_exception.exceptions import ClientException, ServerException
from aliyunsdkcore.client import AcsClient
from aliyunsdkecs.request.v20140526.AuthorizeSecurityGroupRequest import (
    AuthorizeSecurityGroupRequest,
)
from aliyunsdkecs.request.v20140526.RevokeSecurityGroupRequest import (
    RevokeSecurityGroupRequest,
)

FORMAT = "%(asctime)-15s %(levelname)s %(message)s"
logging.basicConfig(level=logging.INFO, format=FORMAT)

# 修改为自己的 ram 账号
client = AcsClient(
    "x", "x", "x"
)


def get_ip():
    url = "http://www.httpbin.org/ip"
    res = requests.get(url)

    data = res.json()
    return data["origin"] + "/24"


def del_group(security_group_id, ip_protocol, port_range, source_cidr_ip):
    """删除规则"""
    request = RevokeSecurityGroupRequest()
    request.set_accept_format("json")
    request.set_SecurityGroupId(security_group_id)
    request.set_PortRange(port_range)
    request.set_IpProtocol(ip_protocol)
    request.set_SourceCidrIp(source_cidr_ip)

    response = client.do_action_with_exception(request)
    # print(str(response, encoding='utf-8'))


def add_group(security_group_id, ip_protocol, port_range, dsecription, source_cidr_ip):
    """添加规则"""
    request = AuthorizeSecurityGroupRequest()
    request.set_accept_format("json")

    request.set_SecurityGroupId(security_group_id)
    request.set_IpProtocol(ip_protocol)
    request.set_PortRange(port_range)
    request.set_Description(dsecription)
    request.set_SourceCidrIp(source_cidr_ip)

    response = client.do_action_with_exception(request)
    # print(str(response, encoding='utf-8'))


def main():
    security_group_id = "修改为自己的安全组id"
    ip_protocol = "tcp"
    port_range = "9000/9999"
    dsecription = "公司ip"

    old_ip = "192.168.0.1/24"
    new_ip = ""

    while True:
        try:
            new_ip = get_ip()
        except:
            time.sleep(5)
            continue

        if old_ip != new_ip:
            logging.info(f"{old_ip} => {new_ip}")
            del_group(
                security_group_id=security_group_id,
                ip_protocol=ip_protocol,
                port_range=port_range,
                source_cidr_ip=old_ip,
            )
            add_group(
                security_group_id=security_group_id,
                ip_protocol=ip_protocol,
                port_range=port_range,
                dsecription=dsecription,
                source_cidr_ip=new_ip,
            )
            old_ip = new_ip

        time.sleep(60)


if __name__ == "__main__":
    main()

```
