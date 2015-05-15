nginx wiki 中文站：http://wiki.nginx.org/Chs
添加用户和组
groupadd www  
useradd -g www -M www  
1.安装nginx所需的pcre库
tar zxvf pcre-7.8.tar.gz  
cd pcre-7.8/  
./configure  
make && make install  
cd ../ 
2、安装Nginx

tar zxvf nginx-1.0.4.tar.gz  
cd nginx-1.0.4/  
./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module  
make && make install  
cd ../ 

#!/bin/bash  
# nginx Startup script for the Nginx HTTP Server  
#  
# chkconfig: - 85 15  
# description: Nginx is a high-performance web and proxy server.  
# It has a lot of features, but it's not for everyone.  
# processname: nginx  
# pidfile: /var/run/nginx.pid  
# config: /usr/local/nginx/conf/nginx.conf  
nginxd=/usr/local/nginx/sbin/nginx  
nginx_config=/usr/local/nginx/conf/nginx.conf  
nginx_pid=/usr/local/nginx/nginx.pid  
 
RETVAL=0  
prog="nginx" 
 
# Source function library.  
. /etc/rc.d/init.d/functions  
 
# Source networking configuration.  
. /etc/sysconfig/network  
 
# Check that networking is up.  
[ ${NETWORKING} = "no" ] && exit 0  
 
[ -x $nginxd ] || exit 0  
 
 
# Start nginx daemons functions.  
start() {  
 
if [ -e $nginx_pid ];then 
   echo "nginx already running...." 
   exit 1  
fi  
 
   echo -n $"Starting $prog: " 
   daemon $nginxd -c ${nginx_config}  
   RETVAL=$?  
   echo  
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx  
   return $RETVAL  
 
}  
 
 
# Stop nginx daemons functions.  
stop() {  
        echo -n $"Stopping $prog: " 
        killproc $nginxd  
        RETVAL=$?  
        echo  
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid  
}  
 
 
# reload nginx service functions.  
reload() {  
 
    echo -n $"Reloading $prog: " 
 $nginxd -s reload  
    #if your nginx version is below 0.8, please use this command: "kill -HUP `cat ${nginx_pid}`" 
    RETVAL=$?  
    echo  
 
}  
 
# See how we were called.  
case "$1" in 
start)  
        start  
        ;;  
 
stop)  
        stop  
        ;;  
 
reload)  
        reload  
        ;;  
 
restart)  
        stop  
        start  
        ;;  
 
status)  
        status $prog  
        RETVAL=$?  
        ;;  
*)  
        echo $"Usage: $prog {start|stop|restart|reload|status|help}" 
        exit 1  
esac  
 
exit $RETVAL 
保持文件后
 [root@localhost /]# cd /etc/rc.d/init.d
[root@localhost init.d]#  chmod +x nginx
 [root@localhost init.d]# /sbin/chkconfig --level 345 nginx on
任何位置都能运行   service nginx start           可选  start | stop | restart | reload | status |  help