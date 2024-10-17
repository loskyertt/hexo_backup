---
title: Python 获取当前代理 IP
date: 2024-09-21 15:57:38
tags:
    - "python"
excerpt: "主要用于在爬虫中进行 IP 切换"
categories: "Python技巧"
---


# 1.Linux 下查看当前代理 IP

## 1.1 通过命令行查看当前的外部 IP

使用 `curl` 命令向外部网站请求，来查看当前代理访问的外部IP地址：
```bash
curl -x http://localhost:port https://api.ipify.org
```

或使用 `ifconfig.me` 服务：
```bash
curl -x http://localhost:port http://ifconfig.me
```

这里 `localhost:port` 是你本地当前代理运行的地址和端口。

如果你的代理使用的是 `socks` 协议，可以这样：
```bash
curl --socks5 localhost:port https://api.ipify.org
```

## 1.2 通过浏览器插件或在线服务

你可以使用浏览器访问以下在线服务来查看当前外部 IP：
- [What Is My IP](https://www.whatismyip.com/)
- [IPinfo.io](https://ipinfo.io/)

这些网站可以直接显示你当前使用的公网IP地址。如果你使用代理，显示的会是代理服务器的IP。

# 2. Python 获取当前代理 IP

```Python
import re
import requests

# 代理地址，设置为你本地代理的地址和端口
proxy = {
    # 记得改成自己的端口号
    'http': 'http://localhost:7890',
    'https': 'http://localhost:7890',
}

# 访问外部API，获取代理IP
try:
    response = requests.get('http://api.ipify.org', proxies=proxy)
    current_ip = response.text
    print(f"Current proxy IP: {current_ip}")
except requests.RequestException as e:
    print(f"Error retrieving IP: {e}")

# 正则表达式提取代理IP和端口
proxy_address = proxy['http']
match = re.match(r'http://([^\:]+):(\d+)', proxy_address)

if match:
    ip = match.group(1)  # 提取IP地址
    port = match.group(2)  # 提取端口号
    print(f"IP: {ip}, Port: {port}")
else:
    print("Invalid proxy format")
```

如果是在 Docker 容器中运行这段代码，记得把`localhost`改为自己宿主机的`ip`地址。