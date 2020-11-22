# docker 学习

## docker 备份和还原

* 备份
```Shell
# eg.
docker ps && docker commit -p dc2d4ac2f482 hub.agoralab.co/adc/mini_app/mini_app_compiling:20201121_ffmpeg                                 
docker images && docker save -o /Users/szw/believe/docker/bak/20201121_ffmpeg.tar hub.agoralab.co/adc/mini_app/mini_app_compiling:20201121_ffmpeg
# or 注：别人的仓库没有权限
docker tag 167ae3cd4957 ubuntu:14.04_test #给imageID=a25ddfec4d2a打个标签，命名要和服务器一致，tag标签版本号可以不一致
# del tag
docker rmi ubuntu:14.04_test
docker images &&   docker push hub.agoralab.co/adc/mini_app/mini_app_compiling:20201119_ffmpeg #推送到服务器 ，名称要和服务器一致
```

* 还原
```Shell
# eg.
docker pull ...
docker load -i /Users/szw/believe/docker/bak/20201121_ffmpeg.tar
docker images && docker run -d -p 80:80 hub.agoralab.co/adc/mini_app/mini_app_compiling:20201121_ffmpeg
```
