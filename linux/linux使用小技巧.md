# linux 使用小技巧

## sudo 免输入密码
sudo的执行顺序是先到/etc/sudoers 中查找用户是否有权限执行sudo命令，如果用户有权限执行则要求用户输入**自己的密码（非root密码）**然后执行
如果想免输入密码只要修改/etc/sudoers文件
```
# 一般登录用户默认在admin组中
%admin ALL=(ALL) NOPASSWD: ALL
```