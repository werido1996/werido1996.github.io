---
layout: mypost
title: Redis多线程批量检测脚本
categories: [About]
---

# 

## Code

```javascript
#! /usr/bin/env python
# _*_  coding:utf-8 _*_
import socket
import sys,re
from multiprocessing import Pool
from multiprocessing.dummy import Pool as ThreadPool
PASSWORD_DIC=['redis','root','oracle','password','p@aaw0rd','abc123!','123456','admin']
def check(ip):
    port=6379
    timeout = 10
    ip1 = re.findall(r"\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b",ip)
    ip=ip1[0]
    try:
        socket.setdefaulttimeout(timeout)
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((ip, int(port)))
        s.send("INFO\r\n")
        result = s.recv(1024)
        if "redis_version" in result:
            success = open("success.txt","a+")
            success.write(ip+"\n")
            print u"[*] %s 未授权访问" % ip
            success.close()
        elif "Authentication" in result:
            for pass_ in PASSWORD_DIC:
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                s.connect((ip, int(port)))
                s.send("AUTH %s\r\n" %(pass_))
                result = s.recv(1024)
                if '+OK' in result:
                    wpassword = open("wpassword.txt","a+")
                    wpassword.write(ip+"|"+pass_+"\n")
                    print u"[*] %s 存在弱口令，密码：%s" % (ip,pass_)
                    wpassword.close()
    except Exception, e:
        pass
if __name__ == '__main__':
    host=open(sys.argv[1], 'r')
    pool = ThreadPool(10)
    pool.map(check, host)
    pool.close()
    pool.join()

```
