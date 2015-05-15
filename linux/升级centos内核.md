# centos 升级内核

---
## 查看内核版本
```
$ sudo uname -a
> centos 版本信息
```
## 导入public ke
``` shell
$ sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```
## 安装ELRepo到CentOS-6.5中
```
$ sudo rpm -ivh http://www.elrepo.org/elrepo-release-6-5.el6.elrepo.noarch.rpm
```
##  安装kernel-lt
```
$ sudo yum --enablerepo=elrepo-kernel install kernel-lt -y
```
## 修改默认的启动内核,设置default=0
``` 
$ sudo vi /etc/grub.conf
/etc/grub.conf > default=0
```
## 重启，查看内核是否升级成功
```
$ reboot
$ sudo uname -a
> centos 版本信息
```