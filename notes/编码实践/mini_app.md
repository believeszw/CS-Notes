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
* ~docker 的安装和配置(不影响打包) // done~

  * [doc使用手册](https://yeasy.gitbook.io/docker_practice/install/mac)

  * docker login https://hub.agoralab.co/

  * docker pull 镜像

  * docker image ls 查看当前镜像

  * docker run --name miniapp -itd -v /Users/szw/believe/project/bitbucket/cloudPlatform:/mnt hub.agoralab.co/adc/mini_app/build:64-miniapp 会生成一个类似 id 下面用

  * 进入后台 docker exec -it 1ca7768bb30de1fd885c24681f7593f336d36e358f2a9944bf21a92436e25e8c（miniapp） bash

  * vim /etc/apt/sources.list 修改源

  * apt update && apt install cmake && apt install wget && apt-get install uuid-dev && apt-get install libtool libtool-bin && apt install autoconf autogenapt upgrade && cmake --version 版本是不是 3.0 以上

* ~后台代码环境配置 // done~
  ```cpp
    目录结构
    media_build
    --media_server_library
    --AgoraDynamicKey
    # 运行脚本
    $ ./setup_env.sh --no-auto-install  64
    # gperf会报错，autogen.sh 脚本不存在，需要单独从原始 github 仓库下载，解压，运行这个脚本，然后在继续 py 脚本后续的工作
    # 环境配置好后可以写一个 helloworld 测试下
    $ mkdir sketchpad
    $ vim hello.cpp
    // Copyright (c) 2014-2020 Agora.io, Inc.
    // A Hello application built on naive Agora build system

    #include "media_server_library/logging/safe_log.h"

    namespace sketchpat {

    extern "C" int main(int argc, char* argv[]) {
        (void)argc;
        (void)argv;

        SAFE_LOG2(Info) << "Hello world";
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
  执行 ./build.sh
  登陆 https://braum.agoralab.co/luna/?_=1604571260 然后 docker pull hub.agoralab.co/uap/mini_app/mini_app-worker:release_20201111_5_1a124caa0105
  docker tag hub.agoralab.co/uap/mini_app/mini_app-worker:release_20201111_5_1a124caa0105 hub.agoralab.co/uap/mini_app/mini_app-worker:latest
  docker images | grep latest
  docker ps|grep mini_app.worker|awk '{print $1}'|xargs docker rm -f 删除原来在运行的镜像，他会自动拉取最新的
  docker ps 查看正在运行的ports
  # 查看日志
  cd /data/uap/mini_app/log
  grep "sdktest" mini_app.worker.log
  grep "mini_app.worker\[7775\]" worker-manager.log
  # end
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
