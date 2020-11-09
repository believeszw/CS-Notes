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
# 11.09 分割线

  * srs 从 1.x 升级到 3.x
