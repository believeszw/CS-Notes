# FFmpeg 编译合集

## 问题合集

### bitbucket clone 不了
```Shell
# git clone https://bitbucket.org/multicoreware/x265_git 在源中添加
deb http://security.ubuntu.com/ubuntu trusty-security main
deb http://cz.archive.ubuntu.com/ubuntu trusty main universe
```

### ERROR: libxxx not found

```Shell
未安装或未指定 -lxxx，检查是否存在
```

### C compiler test failed.

解决办法：
  难道你真的没有安装gcc？（使用sudo apt-get install gcc解决）不会吧不会吧，还真是因为没有安装gcc。
  如果你不是因为没有安装gcc导致的问题，那么你很可能和我一样，某个地方写错了导致gcc编译没过去，你可以查看misc_src/ffmpeg-4.2.2/ffbuild/config.log文件，看下你哪里写错了。相信你可以的。
  或者使用 clang 编译

### xxx not found using pkg-config

```Shell
export PKG_CONFIG_PATH=/mnt/media_build/media_server_miniapp/srs/objs/ffmpeg.sr
c/_release/lib/pkgconfig
```

### srs 编译 librtmp 库
修改 Makefile ，链接库的顺序不能错，否则会报错
```Shell
# for x86/x64 platform
ifeq ($(GCC), gcc)
    EXTRA_CXX_FLAG = -g -O0 -ldl -lstdc++ -lpthread -L$(SRS_OBJS)/ffmpeg.src/_release/lib/. -lavformat -lavcodec -lavutil -lbz2 -lm -lz -lswresample -lfdk-aac -lx264 -lmp3lame -lspeex
```
