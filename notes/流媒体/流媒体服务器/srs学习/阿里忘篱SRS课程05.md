# [05 课 11.25.03:00](https://www.bilibili.com/video/BV1mD4y1S7jy)

# SRS 开发

## Log

### 基于连接的可分离日志

```Shell
szw@nxiot-server-02:~/my_proj/srs/trunk$ ./objs/srs -c conf/console.conf
[2020-11-25 02:18:33.842][Trace][27273][396] server main cid=396, pid=27273, ppid=27211, asprocess=0
# endline
```
* 颜色区分日志级别

* 进程 pid 27273

* 服务器本身 id 396 ，用 ffmpeg 推流后会产生一个新的 id

* err 日志，当我们用 ffmpeg 推重复的流，server 会有 err 级别的日志产生，可以看到程序的堆栈信息，方便快速定位问题
```Shell
[2020-11-25 02:22:08.610][Error][27273][403][11] serve error code=1028 : service cycle : rtmp: stream service : rtmp: stream /live/livestream is busy
thread [27273][403]: do_cycle() [src/app/srs_app_rtmp_conn.cpp:210][errno=11]
thread [27273][403]: service_cycle() [src/app/srs_app_rtmp_conn.cpp:399][errno=11]
thread [27273][403]: acquire_publish() [src/app/srs_app_rtmp_conn.cpp:931][errno=11](Resource temporarily unavailable)
# endline
```

### 错误日志、错误码和堆栈

```Shell
[2020-11-25 02:22:08.610][Error][27273][403][11] serve error code=1028 : service cycle : rtmp: stream service : rtmp: stream /live/livestream is busy
thread [27273][403]: do_cycle() [src/app/srs_app_rtmp_conn.cpp:210][errno=11]
thread [27273][403]: service_cycle() [src/app/srs_app_rtmp_conn.cpp:399][errno=11]
thread [27273][403]: acquire_publish() [src/app/srs_app_rtmp_conn.cpp:931][errno=11](Resource temporarily unavailable)
# endline
```
刚刚产生的 err 日志，会有详细的信息，包括错误码，可以到 **src/kernel/srs_kernel_error.hpp** 去查看产生的错误吗的原因，这里的 1028 对应 **#define ERROR_SYSTEM_STREAM_BUSY 1028**，还有堆栈的信息，对应的文件和行数


### 崩溃时的 coredump

```Shell
szw@nxiot-server-02:~/my_proj/srs/trunk$ ulimit -a
core file size          (blocks, -c) 0   # 关注下这一行，需要 ulimit -c unlimited 打开
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 127892
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 127892
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
拿到 core 文件后，执行对应的可执行文件
```Shell
gdb objs/srs -c core.29487
bt 打印堆栈
#1  0x000000000...
#2  0x........
#3  0x........
f 3 可以进入到第三行函数
p 打印局部变量
```

### WebRTC 客户端日志

```Shell
在 chrome 地址栏输入 chrome://webrtc-internals
一般可以找到 xxRTPVideoStream ，点击查看详细信息

```
