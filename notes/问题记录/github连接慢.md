# git 连接速度慢

## 设置代理

## 修改 host
git clone特别慢是因为github.global.ssl.fastly.net域名被限制了。

因此，只要找到你当前线路最快的ip，修改一下host就能提速。

在网站 https://www.ipaddress.com 分别找这两个域名所对应的最快IP地址

```Cpp
github.global.ssl.fastly.net
github.com
```

在host里面做一下映射就好了。

example
```Cpp
151.101.72.249 github.http://global.ssl.fastly.net
192.30.253.112 github.com
```
