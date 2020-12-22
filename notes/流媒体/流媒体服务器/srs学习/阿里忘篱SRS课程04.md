# [04 课 11.13](https://www.bilibili.com/video/BV1az4y1Q7zL/?spm_id_from=trigger_reload)

# SRS 开发

## SRS 配置

### 配置文件结构

* conf 目录下根据场景有很多对应的配置文件

* 开发时将日志打印在控制台上 ./objs/srs -c conf/console.conf

* 查看 console.conf 文件
```Cpp
cat conf/console.conf
# no-daemon and write log to console config for srs.
# @see full.conf for detail config.
listen              1935;
max_connections     1000;
daemon              off;
srs_log_tank        console;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
}
vhost __defaultVhost__ {
}
```

* srs 默认配置文件，docker 中也会默认使用这个 --- conf/srs.conf
```Cpp
[root@48891d1136d8 trunk]# cat conf/srs.conf
# main config for srs.
# @see full.conf for detail config.
listen              1935;
max_connections     1000;
srs_log_tank        file;
srs_log_file        ./objs/srs.log;
daemon              on;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}
stats {
    network         0;
    disk            sda sdb xvda xvdb;
}
vhost __defaultVhost__ {
    hls {
        enabled         on;
    }
    http_remux {
        enabled     on;
        mount       [vhost]/[app]/[stream].flv;
    }
}
```

* srs 所有配置项 conf/full.conf

* srs 服务器 main 函数 ----  src/main/srs_main_server.cpp

* main 中有这么一个函数 SrsConfig::parse_options 可以通过 gdb 来行进调试
```CPP
[root@48891d1136d8 trunk]# gdb --args ./objs/srs -c conf/console.conf
[root@48891d1136d8 trunk]# b SrsConfig::parse_options
```

### 隔离，应用到不同的客户

*  通过 vhost 实现不同的流用不同的配置
```Cpp
[root@48891d1136d8 trunk]# vim conf/console.conf
...
http_server {
  ...
}
vhost sport.ossrs.net {
  drv {
    ...
  }
}
vhost tv.orrsr.net {
  hls {
    ...  
  }
  drv {
    ...
  }
}
```

* 推流的时候指定 vhost
```Cpp
// err serve error code=2014 : service cycle : rtmp : stream service : check vhost : rtmp : nohost 127.0.0.1
[root@48891d1136d8 trunk]# ffmpeg -re -i ./doc/source.200kbps.768x320.flv -c copy -f flv -y rtmp://127.0.0.1/live/livestream
// yes 会推到 vhost=sport.ossrs.net
[root@48891d1136d8 trunk]# ffmpeg -re -i ./doc/source.200kbps.768x320.flv -c copy -f flv -y rtmp://127.0.0.1/live/livestream?vhost=sport.ossrs.net
// 如果推到 tv 的话，则录制和 hls 都会启用
```

* 参数解析

```Shell
-re：ffmpeg 默认打开文件以后，处理速度很快。为了让 ffmpeg 模拟本地设备，以流的速度处理，需要此选项。
-i：表示输入视频文件，后跟视频文件路径/URL。
-f：这是输出格式，我们这里用 -f flv 标明需要通过 flv 格式推流。
-c：这是推流的过程中，直接 copy 编码的 H264/AAC 流，避免解码、编码。
-vcodec：指定视频解码器，v是视频video，codec就是解码器，后跟解码器名称，copy表示不作解码；
-acodec：指定音频解码器，同理，a是audio，后跟解码器名称。an代表acodec none就是去掉音频的意思。
```

### Reload 不重启服务生效配置

* 输入 killall -1 srs 就会 reload conf  

* 可以修改 conf/console.conf 中 srs_log_tank 对应值 为 file ，再运行上述命令，可以看到输出到了文件中去

* 可以关注并调试 SrsFastLog::on_reload_log_tank()

### 新增自己的配置

* 参考 srs_app_config.cpp
