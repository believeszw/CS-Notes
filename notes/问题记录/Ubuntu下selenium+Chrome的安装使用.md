# Ubuntu下selenium+Chrome的安装使用

## 安装 chrome
* 官网下载安装包

* `sudo dpkg -i google-chrome-stable_current_amd64.deb`

* `whereis google-chrome`

## 安装selenium
`pip3 install selenium`

## 下载chromedriver(火狐使用geckodriver)驱动
[http://npm.taobao.org/mirrors/chromedriver/](http://npm.taobao.org/mirrors/chromedriver/)

下载后,将 `chromedriver` 文件放到 `/usr/bin` 下

`sudo mv chromedriver路径 /usr/bin`
