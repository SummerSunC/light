在linux的内核参数调整中,有几个参数是可以调整下的,比如用netstat发现 
如下很多time-wait数量 
```
netstat -ae|grep 1521|grep root 
```
通过查看:vi /etc/sysctl.conf 
可以看到 
```
#表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。 
net.ipv4.tcp_fin_timeout = 30 
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.tcp_keepalive_time = 1200 
#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭； 
net.ipv4.tcp_syncookies = 1 
#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭； 
net.ipv4.tcp_tw_reuse = 1 
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。 
net.ipv4.tcp_tw_recycle = 1 
#表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。 
net.ipv4.ip_local_port_range = 1024    65000 
net.ipv4.tcp_max_syn_backlog = 8192 
#表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。默认为180000，改为5000。
net.ipv4.tcp_max_tw_buckets = 5000 
```

## 推荐
```
sysctl net.ipv4.tcp_max_tw_buckets=6000
sysctl net.ipv4.ip_local_port_range="1024 65000"
sysctl net.ipv4.tcp_tw_recycle =1
sysctl net.ipv4.tcp_tw_reuse=1
sysctl net.ipv4.tcp_syncookies=1
sysctl net.core.somaxconn=262144
sysctl net.core.netdev_max_backlog=262144
sysctl net.ipv4.tcp_max_orphans=262144
sysctl net.ipv4.tcp_max_syn_backlog=262144
sysctl net.ipv4.tcp_synack_retries=1
sysctl net.ipv4.tcp_syn_retries=1
sysctl net.ipv4.tcp_fin_timeout=1
sysctl net.ipv4.tcp_keepalive_time=30
sysctl fs.file-max=262144
```

- 对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于Squid，效果却不大。此项参数可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。 
当TIME_WAIT太多时，通过dmsg可以看到，会出现TCP: time wait bucket table overflow这样的错误。



## 生效配置
```
/sbin/sysctl -p
```