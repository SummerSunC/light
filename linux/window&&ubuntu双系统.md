# window && ubuntu双系统

##参考
[使用EasyBCD安装Win7和Ubuntu双系统](http://blog.sina.com.cn/s/blog_63c993300101fnzu.html)

## 准备
- 默认在windows环境下
- 安装easyBCD
- 下载操作系统iso文件
- 腾出空闲磁盘，并删除磁盘

## 开始
将iso文件和iso文件中的\casper\vmlinuz 和\casper\initrd.lz放到C盘根目录

使用easyBCD添加磁盘引导NeoGrub引导。点击安装后再点击配置，
在配置文件中加入
```
title Install Ubuntu 10.10
root (hd0,0)
kernel (hd0,0)/vmlinuz boot=casper iso-scan/filename=/ubuntu-10.10-desktop-i386.iso ro quiet splash locale=zh_CN.UTF-8
initrd (hd0,0)/initrd.lz
```
## 安装ubuntu
运行
```
sudo umount -l /isodevice
```
点击安装ubuntu
重启电脑

## 设置引导
进入windows
打开easyBCD 删除NeoGrub引导，新建一个linux/BSD引导
重启

## 收尾
删除不需要的文件