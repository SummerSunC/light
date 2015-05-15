# Redis常用数据结构和操作

---

## String 字符类型
>        Set name luowen  设置name = luowen 存储
>        Get name          获取设置好的name的值
>
>        Setnx name luowen 设置name键值为luowen 如果存在,则返回0 不存在返回1
>        Mset name luowen age 23 salary 233333 设置多个键值对 一块存错 全成功,全失败
>        Msetnx name maomao age 23 hoby basketball 如果设置多个键值对中有存在返回失败
>        Mget name age salary 获取多个键的值
>
>        Getset name maomao 获取name的值,并设置新的值为maomao
>
>        Setrange name 3 maomao 将键name 3字符和面的进行替换 结果为luomaomao
>        Getrange name 3 6 获取键name的值 结果为luomaomao
>
>        Append name .com 给键nane追加.com 结果为luowen.com 
>
>        Incr age 设置每个值自增 返回结果为24
>        Incrby age 6 给name加上6 如果是负数则键
>
>        Decr 与incr相反
>        Decrby 与decrby相反

>        Strlen 返回键对应的值得字符长度


## Hash 键值对 

>        Hset user:001 name luowen    设置哈表名字user 表里面的001 的name 设置为 luowen
>        Hsetnx user name maomao    设置哈希表名字中的name 存在,设置不成功
>
>        Hget user:001 name 获取hash表的user的001的值
>
>        Hmset user:003 name maomao age 23 批量设置
>        Hmget user:003 name age 批量获取user:003的值
>
>        Hincrby user:003 age 3     给hash表的age值加上3
>
>        Hexists user:003 name 判断hash表中式否存在name的键
>
>        Hlen user:003 返回hash表的所有的字段的数目
>        Hkeys user:003 返回hash表的所有字段
>        Hvals user:003 返回hash表中所有的值
>        Hgetall user:003 返回所有的字段和值
>
>        Hdel user:003 name 对hash的name的值和键删除

## list 链表 (双向链表)
### 栈:先进后出 队列:先进先出
### lpush 从头压入
>        Lpush list1 “world” 
>        lpush list1 ‘hello”
>
>        Lrange list1 0 -1 把链表中的数据从0到尾全部取出
>        Word
>        hello

### rpush 从尾部压入
>       rpush list2 “world” 
>       rpush list2 “luowen” 
>
>        lrange list2 0 -1
>        world
>        luowen
### linsert 插入出入数据
>        Rpush list3 luowen
>        Rpush list3 maomao
>        Lrange list3 0 -1
>        Luowen
>        Maomao
>        
>        Linsert list3 before maomao love
>        Lrange list3 0 -1
>        Luowen
>        Love
>        maomao
>
>
>        Linsert list3 after luowen love
>        Lrange list3 0 -1
>        Luowen
>        Love
>        Maomao

### lset 给某个元素复制
>        Rpush list5 luowen
>        Rpush list5 maomao
>
>        Lset list5 0 “deom”
>
>        Demo
>        maomao 

### lrem 删除list表中的数据
>        Rpush list6 luowen
>        Rpush list6 luowen1
>        Rpush list6 luowen2
>        Rpush list6 luowen3
>        Rpush list6 luowen4
>
>        Lrem list6 1 “luowen”
>        删除list6 中值为luowen的值
### ltrim 
>        Lpush list7 luowen1
>        Lpush list7 luowen2
>        Lpush list7 luowen3
>        Lpush list7 luowen4
>        Lpush list7 luowen5
>
>        Ltrim list7 1 2 (1 2 为保留的范围)
>        Lpush list7 luowen2
>        Lpush list7 luowen3
### lpop 从链表的头部弹出一个元素
>        Lpush list8 luowen1
>        Lpush list8 luowen2
>        Lpush list8 luowen3
>
>        Lpop list8 
>        Lpush list8 luowen2
>        Lpush list8 luowen3
### rpop 从链表的尾部弹出一个元素
>        Lpush list8 luowen1
>        Lpush list8 luowen2
>        Lpush list8 luowen3
>
>        rpop list8 
>        Lpush list8 luowen1
>        Lpush list8 luowen2

### rpoplpush 从一个链表弹出,在从头部压入到另一个链表
        
>       List demo1 
>        Demo1A
>        Demo1B
>        Demo1C
>
>        List demo2
>        Demo2A
>        Demo2B
>        Demo2C
>
>        Rpoplpush demo1 demo2
>
>        List demo1 
>        Demo1A
>        Demo1B
>
>        List demo2
>        Demo1C
>        Demo2A
>        Demo2B
>        Demo2C

### lindex 返回一个list小标的索引值
>        List11
>        one
>        two
>        
>        lindex list11 1(list小标)
>        two
>        lindex list11 0
>        one

### llen 返回这个链表的元素的长度
## set无序集合    
### sadd 向集合中插入一条数据
>        Sadd myset1 luowen

### srem 删除集合中的一个元素
>        Srem myset1 luowen

### smembers 查看集合中的元素
>        Smembers myset1

### spop 从集合随机弹出一个元素,返回键值
### sdiff 两个集合的差集 返回两个集合不一样的,根据第一个集合为标准
>        Setdemo1
>        One
>        two
>        
>        setdemo2
>        one 
>        three    
>        
>        sdiff setdemo1 setdemo2
>        two(与setdemo2不一样)
>        sdiff setdemo2 setdeo1
>        three(与setdemo1 不一样)

### sdiffstroe 将两个差集存储到另外一个集合
        
>        Sdiffstore setdemo1 setdemo2 setdemo3
>        将setdemo1 setdemo2 的差集放到 setdemo3中

### sinter 将两个集合的交集
### sinterstore 将两个集合的交集存储到另外一个集合中
### sunion 将两个集合并集
### sunionstore 将两个集合并集并存储到另外一个集合中
### smove 将以个集合中的元素移动到另外一个集合中
>        Eg smove myset1 mysetA two mysetB 集合中的two元素移动到mysetB中

### scard 查看集合中元素的个数
>        Scard myset1查看myset12元素的个数

### sismember 判断是否是集合中的元素
>        Sismember myset13 luowen 判断luowen是否在myset13中的元素

### srandmember myset14 随机取出myset1 中的元素

### zadd 添加到有序集合中区
>        Zadd myzsent 1 luowen1
>        Zadd myzsent 2 luowen2
>        Zadd myzsent 3 luowen3
>        Zadd myzsent 4 luowen4
>
>        Zrange myzsent 0 -1 withscores

### zrem 删除有序集合中的元素
>        Zrem myzsent luowen1 删除myzsent集合中的luowen1

### zincrby myzsent luowen1 3将myzsent luown1的序号更改为4
>        如果没有,就创建他

### zrank 找到myzsent 对应值得索引
### zrevrank 反过来去索引
### zrangebyscore 返回集合中指定的元素
>        Zrangebyscore mysetdeom 2 5 withscores
>        返回mysetdemo中2-5中的元素

### zcount 返回指定空间的数量
>        Zcount myset 2  4 返回2 4中的元素个数

### zcard 返回集合中所有元素的个数
### zremrangbyrank 删除集合中指定区间的元素,并将索引进行排序
### zremrangbyscore 删除集合中指定元素,按循序进行排序

## Redis常用命令

### Key-values
> keys * 匹配键所有的键. 模糊匹配 keys my* 取出所有已my开头的键
> exists 判断是否键 exists name判断是否有name这个键是否存在
> del 删除键 del name 删除name的键
> expire 设置过期时间 expire key time 
> ttl key 查看键的过期时间
> select database 选择数据库
> move key dababase1 讲key移动dao database1中的数据库中
> persist 取消键的过期时间 
> randomkey 随机返回一个键的值
> rename 重命名一个键
> type key 判断key的数据类型

### Server
> ping ping我们的主机能否链接 链接是否存活
> echo 命令 echo demo直接输出
> select 选择数据库 select 0-16个数据库
> quit exit 退出链接
> dbsize 返回数据库的键的个数
> info 返回服务器相关信息
> config get 返回服务配置信息
> flush db 清空数据库
> flushall 删除所有数据库中所有的键

## Redis 高级应用

- 在配置文件里面设置 requirepass password
- 进入后 auth 密码 进行授权 方法二: 登入或在后面加上 –a 加上密码
- 主从复制:
>   One: 一个master服务器可以拥有多个slave
>   Two: 一个salve可以有多个master 并且还可以与其他的salve相连接
- 配置salve
>   打开salveof注释 并添加主机的ip以及端口
>   主机加了密码的时候还需要配置masterauth 密码
- redis 的事务处理
>        输入:multi 打开一个上下文
>        Set age 10
>        Set age 144
>        -----------------------------------------------------------
>        上面的全部放入队列最后执行
>    Exec 
>        最后age为144
>        回滚
>        Discard    
        
>        Watch 监视键的命令

- Redis的持久化
>   方式一:  snapshotting (快照)将内存的数据写入到文件中 save 500 32     500秒内有32个键发生变化则发起快照到文件中
>   方式二: append only file 将没次写修改的命令保存到文件中
>   配置:打开append only
>        Appendfsync yes
>        Appendfsync always 每次都写入
>        Appendfsync everysec    每个一秒写入
>        Appendfsync no    不写入

- 发布和订阅消息

>   订阅:
>   Subscribe tv1 tv2 订阅了两个频道
>   发布:
>   Publish tv1 luweo
>   注:publish tv1的信息 订阅的信息都可以收到

- 虚拟内存

>   方式一:暂时把不使用的数据放到硬盘里面
>   方式二:可以把数据分割到其他的slave数据服务器中



