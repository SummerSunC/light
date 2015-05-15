## 安装依赖包
> TFS安装通常采用源码编译安装，TFS本身依赖**TFS Common Utils、jemalloc**，由于操作系统是经过定制的，所以gcc等常用的组件是已安装的。
所需软件包包括：**zlib-devellibuuid-devel readline-devel e2fsprogs-devel**

1、yum check-update
2、yum -y install zlib-devel libtool automake libuuid-devel readline-devel e2fsprogs-devel svn git
安装tfs-common utils
3、cd /root/software
解压包 (-C src表示解压到src目录下)
4、tar zxf tfs-common-utils.tar.gz -C src
设置环境变量 [root/software]
5、export TBLIB_ROOT=/opt/tfs
安装tb-common-utils[root/software]
6、cd src/tb-common-utils
7、sh build.sh
安装jemalloc [root/software]
8、tar jxf jemalloc-3.5.1.tar.gz2 -C src
9、cd src/ jemalloc-3.5.1
10、./configure
11、make &&　make install

安装tfs【root/software】
12、tar zxf tfs_stable.tar.gz -C src/
13、cd src/tfs
14、./configure --prefix=/opt/tfs --with-tblib-root=/opt/tfs --without-tcmalloc --with-release
15、make && make install
16、cp /opt/tfs/scripts/tfs /etc/init.d/rc.tfs   ---->TFS服务控制脚本
17、cp conf/ns.conf /opt/tfs/conf/     ----> NameServer服务器执行 【2选一】
17、cp conf/ds.conf /opt/tfs/conf/     ----> DataServer服务器执行 【2选一】
cp conf/ads.conf /opt/tfs/conf/   ----> DataServer服务器执行 【可选】

修改配置文件
18、vi /opt/tfs/conf/ns.conf【nameserver配置】

   注意： 执行fd命令查看磁盘使用情况
#KB, 磁盘实际使用的空间，应该小于df命令Available一项的输出
mount_maxsize=961227000
18、vi /opt/tfs/conf/ds.conf 【dataserver配置】

dataserver 挂载盘
df     //查看挂载硬盘
mkdir -p /data/disk{1,2}
mkfs.ext4 /dev/sda // 请确保你有相应权限
mount -t ext4 /dev/sda /data/disk1
mkfs.ext4 /dev/sdb
mount -t ext4 /dev/sdb /data/disk2
配置开机自动挂载
vi /etc/fstab


如果出错，可以使用umount方法

初始化data server
/opt/tfs/scripts/stfs clear 1-2
/opt/tfs/scripts/stfs format 1-2




启动服务
启动tfs nameserver
/etc/init.d/rc.tfs start_ns
启动tfs data server
/etc/init.d/rc.tfs start_ds 1-2



