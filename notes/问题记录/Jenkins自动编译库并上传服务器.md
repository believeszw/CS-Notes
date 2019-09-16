# Jenkins自动编译库并上传服务器

首先添加 `git` 地址：
<div align="center"> <img src="../pics/2019/Jenkins.png" width="900px"> </div><br>

再添加定时构建，每天夜里构建一次：
<div align="center"> <img src="../pics/2019/Jenkins6.png" width="900px"> </div><br>

执行 `shell` 脚本进行构建
```shell
cd networklayer

echo "build json x86"
cmake -S . -B cmake-build-release -DCMAKE_BUILD_TYPE=Release  -G "CodeBlocks - Unix Makefiles" ./
cd cmake-build-release
make

echo "build json hisi500"
cd ..
cmake -S . -B cmake-build-release-hisi3531 -DCMAKE_C_COMPILER=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/target/bin/arm-hisiv500-linux-gcc -DCMAKE_CXX_COMPILER=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/target/bin/arm-hisiv500-linux-g++ -G "CodeBlocks - Unix Makefiles"
cd cmake-build-release-hisi3531
make


echo "make dir"
if [ ! -d "/var/www/html/libs/networklayer" ]; then
  mkdir /var/www/html/libs/networklayer
fi

if [ ! -d "/var/www/html/libs/networklayer/lib" ]; then
  mkdir /var/www/html/libs/networklayer/lib
fi

if [ ! -d "/var/www/html/libs/networklayer/lib/hisi500" ]; then
  mkdir /var/www/html/libs/networklayer/lib/hisi500
fi

if [ ! -d "/var/www/html/libs/networklayer/lib/x86" ]; then
  mkdir /var/www/html/libs/networklayer/lib/x86
fi

if [ ! -d "/var/www/html/libs/networklayer/include" ]; then
  mkdir /var/www/html/libs/networklayer/include
fi

echo "clean before"
rm -rf /var/www/html/libs/networklayer/include/*
rm -rf /var/www/html/libs/networklayer/lib/hisi500/*
rm -rf /var/www/html/libs/networklayer/lib/x86/*

echo "copy"
cp -dprf ${WORKSPACE}/libs/hisi500/libNetWorkLayer.a  /var/www/html/libs/networklayer/lib/hisi500
cp -dprf ${WORKSPACE}/libs/x86/libNetWorkLayer.a  /var/www/html/libs/networklayer/lib/x86

cp -dprf ${WORKSPACE}/libs/hisi500/libNetWorkLayer.so  /var/www/html/libs/networklayer/lib/hisi500
cp -dprf ${WORKSPACE}/libs/x86/libNetWorkLayer.so  /var/www/html/libs/networklayer/lib/x86

cp -dprf ${WORKSPACE}/networklayer/src/net_work_layer.h /var/www/html/libs/networklayer/include/
cp -dprf ${WORKSPACE}/networklayer/src/net_work_common_data.h /var/www/html/libs/networklayer/include/

ls -lh /var/www/html/libs/networklayer/lib/*
ls -lh /var/www/html/libs/networklayer/include/*

#bak
#cmake -S . -B cmake-build-release-hisi3531 -DCMAKE_C_COMPILER=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/target/bin/arm-hisiv500-linux-gcc -DCMAKE_CXX_COMPILER=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/target/bin/arm-hisiv500-linux-g++ -G "CodeBlocks - Unix Makefiles"
#cd cmake-build-release-hisi3531
#make
```
这里成功添加到了服务器上
<div align="center"> <img src="../pics/2019/Jenkins7.png" width="900px"> </div><br>

然后将本地库上传修改为统一从服务器获取：
```shell
cd mediaService
if [ ! -d "include" ]; then
  mkdir include
fi
cd include
rm -fr *
wget http://192.168.1.132/libs/rtspclient/include/ -r -c -np -nH -E -R html --cut-dirs 3
wget http://192.168.1.132/libs/tulog/include/ -r -c -np -nH -E -R html --cut-dirs 3
wget http://192.168.1.132/libs/networklayer/include/ -r -c -np -nH -E -R html --cut-dirs 3

if [ ! -d "trcode" ]; then
  mkdir trcode
fi
cd trcode
wget http://192.168.1.132/libs/hi3531_video_transcode/include/ -r -c -np -nH -E -R html --cut-dirs 3
cd ../

if [ ! -d "libevent" ]; then
  mkdir libevent
fi
cd libevent
wget http://192.168.1.132/libs/libevent/include/ -r -c -np -nH -E -R html --cut-dirs 3
cd ../

if [ ! -d "json" ]; then
  mkdir json
fi
cd json
wget http://192.168.1.132/libs/json/include/ -r -c -np -nH -E -R html --cut-dirs 3
cd ../

if [ ! -d "gtest" ]; then
  mkdir gtest
fi
cd gtest
wget http://192.168.1.132/libs/gtest/include/ -r -c -np -nH -E -R html --cut-dirs 3
cd ../

cd ../
if [ ! -d "libs" ]; then
  mkdir libs
fi
cd libs
rm -fr *
if [ ! -d "hisi500" ]; then
  mkdir hisi500
fi
cd hisi500
wget http://192.168.1.132/libs/hi3531_video_transcode/lib/ -r -c -np -nH -E -R html --cut-dirs 3
wget http://192.168.1.132/libs/json/lib/hisi500/ -r -c -np -nH -E -R html --cut-dirs 4
wget http://192.168.1.132/libs/networklayer/lib/hisi500/ -r -c -np -nH -E -R html --cut-dirs 4
wget http://192.168.1.132/libs/rtspclient/lib/hisi500/ -r -c -np -nH -E -R html --cut-dirs 4
wget http://192.168.1.132/libs/tulog/lib/hisi500/ -r -c -np -nH -E -R html --cut-dirs 4

if [ ! -d "gtest" ]; then
  mkdir gtest
fi
cd gtest
wget http://192.168.1.132/libs/gtest/lib/hisi500/ -r -c -np -nH -E -R html --cut-dirs 4
cd ../
if [ ! -d "libevent" ]; then
  mkdir libevent
fi
cd libevent
wget http://192.168.1.132/libs/libevent/lib/hisi500/ -r -c -np -nH -E -R html --cut-dirs 4
cd ../
cd ../
cd ../

ls -lh ${WORKSPACE}/mediaService/include/*
ls -lh ${WORKSPACE}/mediaService/libs/*

cmake -S . -B cmake-build-release-hisi3531 -DCMAKE_C_COMPILER=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/target/bin/arm-hisiv500-linux-gcc -DCMAKE_CXX_COMPILER=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/target/bin/arm-hisiv500-linux-g++ -G "CodeBlocks - Unix Makefiles"
cd cmake-build-release-hisi3531
make
```
