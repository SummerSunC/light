
##相关资料汇总
- TFS详解博客：http://blog.yunnotes.net/
- TFS项目地址：http://code.taobao.org/p/tfs/wiki/index/
- TFS部署资料：http://note.youdao.com/share/?id=4afdc38988446474c0378ace6d0527e3&type=note
- TFS安装步骤：http://linuxsogood.org/1061.html
- 分布式文件管理系统分析：http://www.open-open.com/lib/view/open1330605869374.html

TAIR(tair是淘宝开源的分布式key/value存储系统):http://code.taobao.org/p/tair/wiki/intro/

给TFS使用的分区，在format，并且DS启动后，相关的分区都是全部给TFS了，即使DS中，你写入的文件只有1M，那边也是100%
这主要是因为，TFS要预先将所有的block都分配完，而且TFS在ssm中看到的已使用容量在85左右时候，就需要扩容了

## TFS 常用命令
``` shell
 ssm -s 10.1.207.114:8100 -i "server" 
root/software/src/tfs

./configure --prefix=/opt/tfs/ --with-release --without-tcmalloc

openresty
ngx-tfs
yum -y install ruby readline-devel zlib-devel
```

** 监控tfs是否停机查看日志搜多destroyed关键字**

## 监控TFSblock 复制情况
```
ssm -s 10.129.195.52:8100 -i "block" | grep '^.*2$' |wc -l
ssm -s 10.129.195.52:8100 -i "block" | grep -v '^.*[3|S]$' 
 BLOCK_ID VERSION FILECOUNT SIZE DEL_FILE DEL_SIZE SEQ_NO COPYS
ssm -s ns_ip:port -i "server -i 3 -c 0"
```
## 如何进行错误（故障）定位
主要看错误码，tfs/src/common/err_msg.h里有每个错误码对应的含义，通常从错误码就能大致看出问题的，（-1000，0)范围内的错误码是保留的，避免跟系统调用的errno冲突。




