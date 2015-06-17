# python orm框架 sqlAlchemy


---
## 依赖
### 安装mysql驱动
``` shell
$ easy_install mysql-connector-python
#window 用安装版安装 下载地址：http://www.codegood.com/archives/129
$ pip install MySQL-python
```
### 虚拟环境安装
进入虚拟目录 virtualenv\scripts
``` shell
#进入虚拟环境
$ activate
#windows 下如果安装报错，可以
# 1、先将真实环境装好，
# 2、复制real_env/Lib/site-packages下相关mysql文件
# 3、粘贴进 virtual_env/Lib/site-package 目录
$ pip install python-mysqldb
#退出虚拟环境
$ deactivate
```


``` shell
$ pip install sqlalchemy
```

``` python
# -*- coding:utf-8 -*-

from sqlalchemy import Column, String, create_engine,ForeignKey
from sqlalchemy.orm import sessionmaker,relationship,backref
from sqlalchemy.ext.declarative import declarative_base

'''
    模型
'''

# 创建对象的基类
Base = declarative_base()

# User Entity
class User(Base):
    # table name
    __tablename__ = "user"

    # column
    id = Column(String(32),primary_key=True)
    name = Column(String(32))
    accounts = relationship('Account')

class Account(Base):
    # table name
    __tablename__="user_account"

    # colum
    id = Column(String(32),primary_key=True)
    real_name = Column(String(32))
    owner_id = Column(String(32),ForeignKey('user.id'))


# 初始化数据库链接
# '数据库类型+数据库驱动名称://用户名:口令@机器地址:端口号/数据库名'
engine = create_engine("mysql://username:password@host:port/dbname")
#创建DBSession类型
DBSession = sessionmaker(bind=engine)

session = DBSession()

user = session.query(User).filter(User.id=='4151d02a7c4f4851bdfd71747ea01c50').one()

print user.name
print user.accounts[0].id

session.close()
```

    



