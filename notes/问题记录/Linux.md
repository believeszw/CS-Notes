## 目录
* [pip安装失败缺少SOCKS依赖](#pip安装失败缺少SOCKS依赖)
* [VM中与主机共享SS](#VM中与主机共享SS)
* [Linux下安装libsodium启用ss的chacha20高级加密](#启用ss的chacha20高级加密)
* [Linux磁盘空间不够进行扩容](#Linux磁盘空间不够进行扩容)

### pip安装失败缺少SOCKS依赖
报错：`pip install doesnt work , InvalidSchema: Missing dependencies for SOCKS support`
* `export all_proxy="socks5://127.0.0.1:1080/"`
* `unset all_proxy`
* `unset ALL_PROXY`
* `pip install pysocks`
* `printenv | grep -i proxy`查看所有代理
* `export all_proxy="https://127.0.0.1:1080/"`指定代理

### VM中与主机共享SS
* `VM`选择`NAT`模式
* `SS`选择允许其他设备连入
* `Ubuntu`中选择手动代理地址为主机 `IPv4`地址，端口`1080`

### 启用ss的chacha20高级加密
* `apt-get install build-essential`
* `wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz`
* `tar xf XXXXX.tar.gz && cd libsodium-XXXXX`包名
* `./configure && make -j4 && make install`
* `ldconfig`

### Linux磁盘空间不够进行扩容
* `VM`虚拟机上进行磁盘的扩展
* `sudo apt-get install gparted`
* `sudo gparted`
* 进行扩容

### snap安装失败
* `error: snap "clion" has "install-snap" change in progress`
run:
```Cpp
$ snap changes
<snip>
203  Doing   2016-06-24T12:17:45Z  -  Install "nextcloud" snap
$ sudo snap abort 203
```
