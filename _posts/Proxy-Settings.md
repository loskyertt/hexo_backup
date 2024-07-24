---
title: proxy-settings
date: 2024-07-19 03:36:58
tags:
    - "proxy"
excerpt: false
---


# 一、代理软件配置

## 1.1 speedTest

```bash
# 日本
https://185-140-53-2.lg.looking.house/1000.mb
# 新加坡
https://23-27-101-2.lg.looking.house/100.mb

https://speed.cloudflare.com/__down?bytes=10000000
https://speed.cloudflare.com/__down?bytes=50000000
https://speed.cloudflare.com/__down?bytes=100000000
https://speed.cloudflare.com/__down?bytes=200000000

# 老版测速
http://cachefly.cachefly.net/10mb.test
http://cachefly.cachefly.net/50mb.test
http://cachefly.cachefly.net/100mb.test
```

## 1.2 speedurl

```
https://www.google.com/generate_204

http://cp.cloudflare.com/

http://connectivitycheck.gstatic.com/generate_204
```

## 1.3 DNS选项

- **远程DNS**

```bash
udp://1.1.1.1

https://dns.google/dns-query
```

- **直连DNS**

```bash
114.114.114.114

https://doh.pub/dns-query
```
