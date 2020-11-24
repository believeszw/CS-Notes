# [03 课 11.12.02:45](https://www.bilibili.com/video/BV1az4y1Q7zL/?spm_id_from=trigger_reload)

## SRS 开发

### 获取代码

* [github](https://github.com/ossrs/srs)

* gitee

### IDE 开发工具

* IDEA

* VS Code

### 编译调试器

* macOS

  * lldb 调试
```Shell
lldb -- ./objs/srs -c conf/console.conf
```

* CentoS7

* srs-docker/dev

  * gdb 调试
```Shell
gdb --args ./objs/srs -c conf/console.conf
```


### 代码结构

* modules

  * core

  * kernel

  * protocol

  * app

  * main

* server-listener

* source-consumer

### rtmp 推流

* docker ps 可以看到 container id ，docker exec -it (container id) bash

* ps aux|grep srs 查看 pid 和 原来 exec 的 docker 日志中的 pid 对比是否进入的同一个

* ffmpeg 推流

  * 编写SRS配置文件。详细参考RTMP分发\
  将以下内容保存为文件，譬如conf/rtmp.conf，服务器启动时指定该配置文件(srs的conf文件夹有该文件)。
```Shell
# conf/rtmp.conf
listen              1935;
max_connections     1000;
vhost __defaultVhost__ {
}
```
* ./objs/srs -c conf/rtmp.conf
>系统服务，init.d脚本：SRS提供srs/trunk/etc/init.d/srs脚本，可以作为CentOS或者Ubuntu的系统服务自动启动。

* 启动推流编码器
```Shell
ffmpeg -re -i ./doc/source.200kbps.768x320.flv -vcodec copy -acodec copy -f flv -y rtmp://localhost/live/livestream;
# or
ffmpeg -re -i ./doc/source.200kbps.768x320.flv -c copy -f flv -y rtmp://localhost/live/livestream;
```

* 观看RTMP流。我 server 的 IP 地址为 10.80.0.23 则播放地址为 rtmp://10.80.0.23/live/livestream（可以通过 VLC 或者 srs 官方 [player](http://ossrs.net/srs.release/trunk/research/players/srs_player.html?vhost=__defaultVhost__&autostart=true&server=192.168.1.170&app=live&stream=livestream&port=1935)）
```Shell
rtmp://localhost/live/livestream
```  
