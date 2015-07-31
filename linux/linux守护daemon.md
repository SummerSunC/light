# linux 守护进程

在使用ssh登录linux时候运行类似于java -jar 的命令时，程序的输出会占用控制台，
造成无法继续操作，如果关闭会话，则进程也会被杀掉

## start-stop-daemon
[centos安装start-stop-daemon](http://blog.creke.net/776.html)

## nohup
解决这个问题可以使用nohup（不挂断执行命令） + &（后台运行进程） 

```
nohup {command} &
```
