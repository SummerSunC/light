# 安装nginx

## 准备
### 帮助
[nginx 加入pcre库时编译错误](http://www.oschina.net/question/240916_120681?sort=time)

[参考资料](http://nginx.org/en/docs/beginners_guide.html)

[反向代理](http://www.nowamagic.net/academy/detail/1226280)

### 依赖安装的软件：
```
yum -y install cmake ruby git zlib-devel readline-devel gcc gcc-c++ pcre
```
### 增加用户及用户组
```
groupadd www
useradd -g www -s /sbin/nologin -m www
```

### 下载所需包

- yajl(json解析器) : wget -O yajl.tar.gz https://codeload.github.com/lloyd/yajl/legacy.tar.gz/2.0.1
- nginx : wget http://nginx.org/download/nginx-1.4.5.tar.gz
- nginx-tfs : git clone https://github.com/alibaba/nginx-tfs.git
- pcre(c语言正则解析器) :wget  ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.34.tar.gz 


## 解压：

### yajl src:
 ```
 ./configure
 make && make install
 echo "/usr/local/lib" > /etc/ld.so.conf.d/yajl.conf
```

### nginx src:
```
./configure \
--prefix=/opt/nginx \
--user=www \
--group=www \
--without-select_module \
--without-poll_module \
--with-file-aio \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--without-http_ssi_module \
--without-http_autoindex_module \
--without-http_uwsgi_module \
--without-http_scgi_module \
--without-http_memcached_module \
--http-client-body-temp-path=/opt/nginx/cache/client/ \
--http-proxy-temp-path=/opt/nginx/cache/proxy/ \
--without-http_fastcgi_module \
--without-mail_pop3_module \
--without-mail_imap_module \
--without-mail_smtp_module \
--with-pcre=../pcre-8.34 \
--add-module=../nginx-tfs

make && make install
```

## 修改配置

### 修改 ……/conf/nginx.conf
### 增加启动脚本：/etc/init.d/nginx
```
vi /etc/init.d/nginx   （见<./脚本/将nginx添加到系统服务中.md>) 
```
## 服务器设置

### limits.conf
```
vi /etc/security/limits.conf
# 加上
* soft nofile 102400
* hard nofile 102400
```
### 关闭防火墙：
```
service iptables stop
chkconfig 系统服务
```
### 开机启动
```
echo "/etc/init.d/nginx" >> /etc/rc.local
```
## 安装nginx_tfs插件（可选）
### 下载nginx_tfs
```
git clone https://github.com/alibaba/nginx-tfs.git
        http://openresty.org/
wget http://openresty.org/download/ngx_openresty-1.5.8.1.tar.gz
tar xvzf ngx_openresty-1.5.8.1.tar.gz
cd ngx_openresty-1.5.8.1
./configure --prefix=/opt/nginx --add-module=/root/software/src/nginx-tfs
```



