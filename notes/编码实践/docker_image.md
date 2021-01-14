# image 环境安装

* cmake
```Shell
sudo apt-get install build-essential
sudo apt-get install libssl-dev
wget http://www.cmake.org/files/v3.16/cmake-3.16.0.tar.gz
tar xf cmake-3.16.0.tar.gz && cd cmake-3.16.0 && ./configure && make -j16 && sudo make install
sudo vim ~/.bashrc
export PATH=/usr/local/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
source ~/.bashrc && cmake --version
```

* shadowsocks
```Shell
# install
sudo apt-get install python
sudo apt-get install python-pip
sudo pip install shadowsocks
# config
mkdir -p /home/${user}/shadowsocks/shadowsocks.json
{
  "server": "{your-server}",
  "server_port": 40002,
  "local_port": 1080,
  "password": "{your-password}",
  "timeout": 600,
  "method": "aes-256-cfb"
}
sudo sslocal -c /home/szw/shadowsocks/shadowsocks.json -d start
# proxy
sudo apt-get install polipo
vim /etc/polipo/config
logSyslog = true
logFile = /var/log/polipo/polipo.log
proxyAddress = "0.0.0.0"
socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5
chunkHighMark = 50331648
objectHighMark = 16384
serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
sudo /etc/init.d/polipo restart
# start
sudo sslocal -c /home/shadowsocks/shadowsocks.json -d start
export http_proxy="http://127.0.0.1:8123/"
curl www.google.com
```

* 配置 git 代理

```Shell
# git config --global http.proxy 'socks5://127.0.0.1:8123'
# git config --global https.proxy 'socks5://127.0.0.1:8123'
git config --global http.proxy 'http://127.0.0.1:8123'
git config --global https.proxy 'http://127.0.0.1:8123'
注意端口号也设置代理的端口号，不同vpn，不同系统端口号可能不同。
```

* ffmpeg

```Cpp
# Get the Dependencies
apt-get update -qq && apt-get -y install \
  git  \
  aptitude  \
  libsdl2-2.0-0  \
  libegl1-mesa-dev  \
  libgles2-mesa-dev  \
  libglu1-mesa-dev  \
  autoconf \
  automake \
  build-essential \
  cmake \
  git-core \
  libass-dev \
  libfreetype6-dev \
  libgnutls28-dev \
  libsdl2-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libxcb1-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  pkg-config \
  texinfo \
  wget \
  yasm \
  lzma \
  libbz2-dev \
  liblzma-dev \
  zlib1g-dev
# out_dir
mkdir -p /home/ffmpeg/ffmpeg_sources /home/bin

# NASM
apt-get install nasm
# or
cd /home/ffmpeg/ffmpeg_sources && \
wget https://www.nasm.us/pub/nasm/releasebuilds/2.14.02/nasm-2.14.02.tar.bz2 && \
tar xjvf nasm-2.14.02.tar.bz2 && \
cd nasm-2.14.02 && \
./autogen.sh && \
PATH="/home/bin:$PATH" ./configure --prefix="/home/ffmpeg/ffmpeg_build" --bindir="/home/bin" && \
make -j16 && \
make install

# libx264
apt-get install libx264-dev
# or
cd /home/ffmpeg/ffmpeg_sources && \
git -C x264 pull 2> /dev/null || git clone --depth 1 https://code.videolan.org/videolan/x264.git && \
cd x264 && \
PATH="/home/bin:$PATH" PKG_CONFIG_PATH="/home/ffmpeg/ffmpeg_build/lib/pkgconfig" ./configure --prefix="/home/ffmpeg/ffmpeg_build" --bindir="/home/bin" --enable-static --enable-pic && \
PATH="/home/bin:$PATH" make -j16 && \
make install

# libx265
apt-get install libx265-dev libnuma-dev
# or
apt-get install libnuma-dev && \
cd /home/ffmpeg/ffmpeg_sources && \
git -C x265_git pull 2> /dev/null || git clone https://bitbucket.org/multicoreware/x265_git && \
cd x265_git/build/linux && \
PATH="/home/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/home/ffmpeg/ffmpeg_build" -DENABLE_SHARED=off ../../source && \
PATH="/home/bin:$PATH" make -j16 && \
make install

# libvpx
apt-get install libvpx-dev
# or
cd /home/ffmpeg/ffmpeg_sources && \
git -C libvpx pull 2> /dev/null || git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git && \
cd libvpx && \
PATH="/home/bin:$PATH" ./configure --prefix="/home/ffmpeg/ffmpeg_build" --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=yasm && \
PATH="/home/bin:$PATH" make -j16 && \
make install

# libfdk-aac
apt-get install libfdk-aac-dev
# or
cd /home/ffmpeg/ffmpeg_sources && \
git -C fdk-aac pull 2> /dev/null || git clone --depth 1 https://github.com/mstorsjo/fdk-aac && \
cd fdk-aac && \
autoreconf -fiv && \
./configure --prefix="/home/ffmpeg/ffmpeg_build" --disable-shared && \
make -j16 && \
make install

# libmp3lame
apt-get install libmp3lame-dev
# or
cd /home/ffmpeg/ffmpeg_sources && \
wget -O lame-3.100.tar.gz https://downloads.sourceforge.net/project/lame/lame/3.100/lame-3.100.tar.gz && \
tar xzvf lame-3.100.tar.gz && \
cd lame-3.100 && \
PATH="/home/bin:$PATH" ./configure --prefix="/home/ffmpeg/ffmpeg_build" --bindir="/home/bin" --disable-shared --enable-nasm && \
PATH="/home/bin:$PATH" make -j16 && \
make install

# libopus
apt-get install libopus-dev
# or
cd /home/ffmpeg/ffmpeg_sources && \
git -C opus pull 2> /dev/null || git clone --depth 1 https://github.com/xiph/opus.git && \
cd opus && \
./autogen.sh && \
./configure --prefix="/home/ffmpeg/ffmpeg_build" --disable-shared && \
make -j16 && \
make install

# libaom
cd /home/ffmpeg/ffmpeg_sources && \
git -C aom pull 2> /dev/null || git clone --depth 1 https://aomedia.googlesource.com/aom && \
mkdir -p aom_build && \
cd aom_build && \
PATH="/home/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/home/ffmpeg/ffmpeg_build" -DENABLE_SHARED=off -DENABLE_NASM=on ../aom && \
PATH="/home/bin:$PATH" make -j16 && \
make install

# libsvtav1
cd /home/ffmpeg/ffmpeg_sources && \
git -C SVT-AV1 pull 2> /dev/null || git clone https://github.com/AOMediaCodec/SVT-AV1.git && \
mkdir -p SVT-AV1/build && \
cd SVT-AV1/build && \
PATH="/home/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/home/ffmpeg/ffmpeg_build" -DCMAKE_BUILD_TYPE=Release -DBUILD_DEC=OFF -DBUILD_SHARED_LIBS=OFF .. && \
PATH="/home/bin:$PATH" make -j16 && \
make install

# lame-3.99
# speex-1.2rc1

# FFmpeg
cd /home/ffmpeg/ffmpeg_sources && \
wget -O ffmpeg-4.1.tar.bz2  https://ffmpeg.org/releases/ffmpeg-4.1.tar.bz2  && \
tar xjvf ffmpeg-4.1.tar.bz2  && \
cd ffmpeg-4.1 && \
PATH="/home/bin:$PATH" PKG_CONFIG_PATH="/home/ffmpeg/ffmpeg_build/lib/pkgconfig" ./configure \
  --enable-gpl --enable-nonfree \
  --prefix="/home/ffmpeg/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I/home/ffmpeg/ffmpeg_build/include" \
  --extra-ldflags="-L/home/ffmpeg/ffmpeg_build/lib" \
  --extra-libs="-lpthread -lm" \
  --bindir="/home/bin" \
  --enable-static \
  --disable-shared \
  --disable-debug \
  --disable-ffplay \
  --disable-ffprobe \
  --disable-doc \
  --enable-postproc  \
  --enable-bzlib \
  --enable-zlib \
  --enable-parsers \
  --enable-libx264 --enable-libmp3lame --enable-libfdk-aac \
  --enable-encoder=libfdk_aac --enable-decoder=libfdk_aac \
  --enable-muxer=adts \
  --enable-pthreads --extra-libs=-lpthread \
  --enable-encoders --enable-decoders --enable-avfilter --enable-muxers --enable-demuxers && \
PATH="/home/bin:$PATH" make -j16 && \
make install && \
hash -r
# 这几个有问题
# --enable-libaom \
# --enable-gnutls \
source ~/.profile
```
