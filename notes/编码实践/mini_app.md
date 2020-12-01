# mini_app 学习

目录结构

media_server_miniapp
|- setup
|- srs // srs 下的 src 下 srs_app_agora 为我们的接口
|- switch wrapper 层 修改这一层
|- test
|- miniapp_workder 业务层主入口

Client/SDK：客户端；
AppCenter：核心服务会分配边缘节点；
WorkerManager：边缘节点管理服务；
Worker：具体业务，实际处理任务；
支撑平台：数据上报、统计、展示以及监控、报警、计费等；

[通用业务平台](file:///Users/szw/believe/project/bitbucket/cloudPlatform/media_build/media_server_uap/docs/docker/html/about/architecture.html)

* mute 相关的在 rtc_connection 类

* 确认 2711 默认是硬编还是软编

* 接收音视频数据回调 SrsAgoraReceiver（onReceiveEncoded）

* 新建分支名 branch:dev/mini-app-1.1

* /Users/szw/believe/project/bitbucket/cloudPlatform/media_build/media_server_miniapp/miniapp_worker/worker.cpp:591 joinchannel

* 确认目前老版本 sdk 具备哪些功能，大重构实现是否有困难
```Shell
# 未使用不需要实现的接口 或 未使用的类
bool IProbeAVFormat(const char* url, IAVFormat &info);
class IAVMapper : public IAVIO;
class IAVAudioMixer : public IAVIO;
class AVAudioMixerEvent;
class IAVRtmpPusher : public IAVIO;
class AVRtmpPusherEvent;
class IAVVideoMixer : public IAVIO;
class IAVCapturer : public IAVIO;
class AVCapturerHandler;
class IAVRecoder : public IAVIO;
class IAVInputer : public IAVIO;
class AVFileReaderEvent;
# 使用需要实现的接口
class IAVEncoder : public IAVIO; // encoder module, wrapper of FFmpeg avcodec. can encode any A/V stream
class IAVDecoder : public IAVIO; // decoder module, wrapper of FFmpeg avcodec. can decode any A/V stream
class AudioConfig;
class VideoConfig;
class AVEncoderEvent; // Callback for insert user defined SEI information
class IAVEngine : public IAVIO; // pull stream from agora cloud or push stream to.
class AVEngineEventHandler; // expose callback from Agora Rtc Engine
class IAVDispatcher; // external A/V source pushing to pipeline
class IAVIO; // base class of module, it define all public fuction for controling. the wrapper of AVIO, which is the internal impl
class IAVHandler; // Callback handle for getting A/V data
class IAVData;
# endline
```

## Miniapp准备工作

* docker 的安装和配置(不影响打包)

  * [doc使用手册](https://yeasy.gitbook.io/docker_practice/install/mac)

  * docker login https://hub.agoralab.co/

  * docker pull 镜像

  * docker image ls 查看当前镜像

  * docker run --name miniapp -itd -v /Users/szw/believe/project/bitbucket/cloudPlatform:/mnt hub.agoralab.co/adc/mini_app/build:64-miniapp 会生成一个类似 id 下面用

  * 进入后台 docker exec -it 1ca7768bb30de1fd885c24681f7593f336d36e358f2a9944bf21a92436e25e8c（miniapp） bash

  * vim /etc/apt/sources.list 修改阿里源，确认以下库有没有安装
```Shell
apt install wget
apt-get install uuid-dev
apt-get install libtool libtool-bin
apt install autoconf autogenapt upgrade
sudo apt install git -y
sudo apt install g++ -y
sudo apt install gcc -y
sudo apt install autoconf -y
sudo apt install libtool -y
sudo apt install cmake3 -y
sudo apt install uuid-dev -y
sudo apt install libevent-dev -y
sudo apt install libsasl2-dev -y
sudo apt install libcrypto++-dev -y
sudo apt install zlib1g-dev -y
sudo apt install libssl-dev -y
sudo apt install libmsgpack-dev -y
sudo apt install libtcmalloc-minimal4 -y
cmake --version 版本是不是 3.0 以上
```
* 目录树
```Shell
.
├── agoratools
│   ├── DynamicKey
│   ├── Golem
│   ├── Proxy
│   └── TroubleShooting
└── media_build
    ├── AgoraDynamicKey -> ../agoratools/DynamicKey/AgoraDynamicKey
    ├── BLADE_ROOT
    ├── build_tools
    ├── extract_lib.sh
    ├── gen_makefile.sh
    ├── media_server_library
    ├── media_server_notification
    ├── media_server_protocol
    ├── media_server_uap
    ├── setup_env.sh
    ├── signaling_common
    ├── signaling_protocol
    └── third_party
```

* 后台代码环境配置
  ```Shell
    git clone ssh://git@git.agoralab.co/cloud/media_build.git
    git clone https://github.i.agoralab.co/AgoraIO/Tools.git
    cd media_build
    ln -s ../Tools/DynamicKey/AgoraDynamicKey AgoraDynamicKey
    ./setup_env.sh 64
    git clone ssh://git@git.agoralab.co/cloud/media_server_uap.git
    # git clone ssh://git@git.agoralab.co/cloud/media_server_notification.git
    # git clone ssh://git@git.agoralab.co/cloud/media_server_protocol.git

    git clone ssh://git@git.agoralab.co/cloud/media_server_miniapp.git
    # git clone git@github.com:AgoraLab/signaling_common.git
    # git clone git@github.com:AgoraLab/signaling_protocol.git
    git clone git@github.com:AgoraLab/media_server_library.git
    # 如果很慢用 git clone ssh://git@git.agoralab.co/cloud/media_server_library.git
    # 如果一定要用 github
    # git clone git@git.zhlh6.cn:AgoraLab/media_server_library.git
    # 然后改 remote 地址
    # cd media_server_library
    # git remote rm origin
    # git remote add origin git@github.com:AgoraLab/media_server_library.git
    # git fetch

    # 运行脚本
    $ ./setup_env.sh 64
    # gperf会报错，autogen.sh 脚本不存在，需要单独从原始 github 仓库下载，解压，运行这个脚本，然后在继续 py 脚本后续的工作   
  ```

* 环境检测

环境配置好后可以写一个 helloworld 测试下

```cpp
$ mkdir sketchpad
$ vim hello.cpp

// Copyright (c) 2014-2020 Agora.io, Inc.
// A Hello application built on naive Agora build system

#include "media_server_library/logging/safe_log.h"

namespace sketchpat {

extern "C" int main(int argc, char* argv[]) {
    (void)argc;
    (void)argv;

    SAFE_LOG2(Info) << "Helloworld";
    return 0;
}

}  // namespace sketchpat
$ vim BUILD
cc_binary(name = "hello.exe",
     srcs = [
              "hello.cpp",
     ],
     cppflags = [
              "-Imedia_server_library",
     ],
     ldflags = [],
     deps = [
              "//media_server_library/logging/BUILD:logging",
     ]
     )
$ ./gen_makefile.sh sketchpad/BUILD -m64
$ make debug -j4
$ .build/x86_64/debug/targets/sketchpad/hello.exe
```


* 小程序发版
```Shell
  cd media_server_miniapp && ./build_release.sh (没有ffmpeg的库不影响)
  ./run_worker.sh && cd conf && python gen.py (第一次需要生成服务器 conf 文件，里面包含服务器的信息)
  ./cp_binary.sh
  exit && 再进入打包发布目录， /Users/szw/believe/project/bitbucket/cloudPlatform/media_build/media_server_miniapp/docker/new/mini_app/app-worker，
  执行 ./build.sh 会生成一个 tag ,下面会用到
  先本地运行
  docker run -itd --net host hub.agoralab.co/uap/mini_app/mini_app-worker:$tag bash
  然后再 exec 运行测试
  # 登陆 https://braum.agoralab.co/luna/?_=1604571260 然后
  docker pull hub.agoralab.co/uap/mini_app/mini_app-worker:$tag && docker tag hub.agoralab.co/uap/mini_app/mini_app-worker:$tag hub.agoralab.co/uap/mini_app/mini_app-worker:latest
  # 将上传的镜像tag更新为latest，程序会自动拉取
  docker pull hub.agoralab.co/uap/mini_app/mini_app-worker:release_20201130_3_6805186d36d9 && docker tag hub.agoralab.co/uap/mini_app/mini_app-worker:release_20201130_3_6805186d36d9 hub.agoralab.co/uap/mini_app/mini_app-worker:latest && ./stop.sh


  # 上次可用的 manager 的镜像
  docker tag hub.agoralab.co/uap/mini_app/mini_app-manager:release_20200831_1_6a1547a33f42 hub.agoralab.co/uap/mini_app/mini_app-manager:latest

  # 原始 release_20201022_4_c10922a0d92c
  hub.agoralab.co/uap/mini_app/mini_app-worker                        release_20201022_8_f990be8ad911   f990be8ad911        3 weeks ago         626MB
  44de568db97d        hub.agoralab.co/uap/mini_app/mini_app-manager:release_20200421_1_5aaa0d834f68   "/usr/src/run_app.sh"   5 hours ago         Up 5 hours                              worker-manager

  docker images | grep latest
  # 删除原来在运行的镜像，他会自动拉取最新的
  docker ps|grep mini_app.worker|awk '{print $1}'|xargs docker rm -f
  # 查看正在运行的
  docker ps
```

* 日志分析
```Shell
# 查看日志
cd /data/uap/mini_app/log
grep "sdktest" mini_app.worker.log 查看 pid
grep "mini_app.worker\[7775\]" worker-manager.log
# 删除日志后需要执行，尽量不要删除日志
sudo service rsyslog restart
# 日志目录下执行，查看更多信息
tail -f worker-manager.log|grep idleW
# 先拷贝，再从文件服务器下载
cp file /tmp/.
# end
# 通过 cname 找到 pid
grep sdktest mini_app.worker.log
output:
2020-11-18 09:02:14 info mini_app.worker[27022]: media_server_library/universal_app/worker_manager_agent.cpp:44: Reveived from worker-manager: {"appId":"0c0b4b61adf94de1befd7cdd78a50444","clientAddress":"rJKDiTVW","clientRequest":{"action":"join","appId":"0c0b4b61adf94de1befd7cdd78a50444","channel_name":"sdktest111816","clientType":"wechat","key_vocs":"0c0b4b61adf94de1befd7cdd78a50444","key_vos":"0c0b4b61adf94de1befd7cdd78a50444","role":"broadcaster","uid":"0"},"cname":"sdktest111816","command":"request","gatewayType":"ws","requestId":3,"sdkAddress":"182.92.60.22:50884","sdkVersion":"1.1.0","seq":2,"sid":"35D38FE97C5141FFA1D3BA86D1BDD391","ts":1605690131,"uid":"0"}
# 通过 pid 查找
grep "\[27022\]" mini_app.worker.log
output:
2020-11-18 09:02:14 info mini_app.worker[27022]: media_server_library/universal_app/worker_manager_agent.cpp:44: Reveived from worker-manager: {"appId":"0c0b4b61adf94de1befd7cdd78a50444","clientAddress":"rJKDiTVW","clientRequest":{"action":"join","appId":"0c0b4b61adf94de1befd7cdd78a50444","channel_name":"sdktest111816","clientType":"wechat","key_vocs":"0c0b4b61adf94de1befd7cdd78a50444","key_vos":"0c0b4b61adf94de1befd7cdd78a50444","role":"broadcaster","uid":"0"},"cname":"sdktest111816","command":"request","gatewayType":"ws","requestId":3,"sdkAddress":"182.92.60.22:50884","sdkVersion":"1.1.0","seq":2,"sid":"35D38FE97C5141FFA1D3BA86D1BDD391","ts":1605690131,"uid":"0"}
# 通过 publish or rtmp 找到 srs 端口
grep publish mini_app.worker.log
output:
2020-11-18 09:18:46 info mini_app.worker[29971]: media_server_library/universal_app/worker_manager_agent.cpp:136: app-worker send message: {"gatewayType":"ws","appId":"e26dbe532a5b44359481cf169ef6eaf5","cname":"fda3c8eb8d3f20bc8b162e2fa127a552","uid":"4330","sid":"3C36F12E4BF94170AE84A1095E73B8D6","seq":2,"ts":1605691126800,"requestId":4,"code":200,"reason":"Success","command":"response","serverResponse":{"action":"publish","url":"rtmp://120.131.14.112:7244/live/oHwVhZplV7LQOER76kQF77VxqGT2omUu"}}
# 通过 端口 查看 srs 日志
 vim miniapp_srs_7244.log
```

* gdb 调试程序
```Shell
# 登陆 web 控制端
docker ps
# 任取一个
4478f4d01dca        hub.agoralab.co/uap/mini_app/mini_app-worker                                    "./run_worker.sh"       About an hour ago   Up About an hour                        mini_app.worker-1605764111947
# 进入镜像
docker exec -it 4478f4d01dca bash
# 查看具体线程
ps aux | grep mini_app
root     31949  0.0  0.0  19124  2392 pts/0    t+   05:35   0:01 ./mini_app.worker -c conf/rtmp-7218.conf
# attach 线程
gdb attach 31949
# bt 可以查看到实时的堆栈信息
#0  0x00007ff7e5fb36b3 in __epoll_wait_nocancel () at ../sysdeps/unix/syscall-template.S:81
#1  0x00000000005a55de in _st_epoll_dispatch () at event.c:1236
#2  0x00000000005a0100 in _st_idle_thread_start (arg=0x0) at sched.c:225
#3  0x00000000005a050b in _st_thread_main () at sched.c:337
#4  0x00000000005a0c7b in st_thread_create (start=0xf4240, arg=0x0, joinable=0, stk_size=5903284) at sched.c:616
#5  0x00000000004be092 in SrsServer::do_cycle (this=0x10538e0) at src/app/srs_app_server.cpp:1129
#6  0x00000000004bda55 in SrsServer::cycle (this=0x10538e0) at src/app/srs_app_server.cpp:997
#7  0x000000000059faa1 in run_master (svr=0x10538e0) at src/main/srs_main_server.cpp:486
#8  0x000000000059f59b in run (svr=0x10538e0) at src/main/srs_main_server.cpp:410
#9  0x000000000059e1e3 in do_main (argc=3, argv=0x7fff18761288) at src/main/srs_main_server.cpp:185
#10 0x000000000059e2e0 in main (argc=3, argv=0x7fff18761288) at src/main/srs_main_server.cpp:193
# 卡在这里
```


* 小程序配置文件在 cat /etc/uap/config/worker-manager.json
```Shell
# keyList 中指聚合的指标，当前是这三个指标匹配后会在次 worker 内
{
  "name": "mini_app",
  "gateway": {
    "wsPort": 7000
  },
  "workerManager": {
    "forWorkerPort": 6001,
    "forGatewayPort": 5002
  },
  "keyList": ["appId", "cname", "uid"],
  "appCenter": {
  },
  "sdk": {
    "aliveTimeoutSeconds": 10,
    "checkConnectionClose": true,
    "fullCloseReport": true
  }
}
```

* 修改 worker-manger 内 start.sh 脚本，修改启动个数，将 50 改成 0 ，然后自己启动 worker ，方便调试


* 一次误操作出现视频，断点发现 srs_app_agora.cpp:497 ，发现 publisher_token_ 被人更新，而 publisher_stream_ 没有同步更新，导致没有画面，定位该问题
```Shell
} else if(publisher_token_ != publish_stream_) {
  return ERROR_CONTROL_RTMP_CLOSE;
}
```
