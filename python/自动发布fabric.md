# fabric

## 传送门

[部署web app](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014323392805925d5b69ddad514511bf0391fe2a0df2b0000)

[Django 部署](http://www.ziqiangxuetang.com/django/django-nginx-deploy.html)

[django 部署02](http://segmentfault.com/q/1010000002523354/a-1020000002526934)

目前fabric只支持python2.x
```python
# -*- coding:utf8 -*-
#  fabfile.py

__author__ = 'suncheng'

import os, re
from datetime import datetime

# 导入Fabric API:
from fabric.api import *
from operator import *

# 服务器登录用户名:
env.user = 'ggyw'
# sudo用户为root:
env.sudo_user = 'root'
# 服务器地址，可以有多个，依次部署:
env.hosts = ['123.59.80.150']
env.password='ggyw!@#456'

# 服务器MySQL用户名和口令:
#db_user = 'www-data'
#db_password = 'www-data'

_TAR_FILE = 'api-test.tar.gz'

def build():
    includes = ['templates', 'demo', 'api_test', 'coupon_api', '*.py', '*.wsgi']
    excludes = ['test', '.*', '*.pyc', '*.pyo']
    local('rm -f dist/%s' % _TAR_FILE)
#    with lcd(os.path.join(os.path.abspath('.'), 'www')):
    cmd = ['tar', '--dereference', '-czvf', 'dist/%s' % _TAR_FILE]
    cmd.extend(['--exclude=\'%s\'' % ex for ex in excludes])
    cmd.extend(includes)
    local(' '.join(cmd))

_REMOTE_TMP_TAR = '/tmp/%s' % _TAR_FILE
_REMOTE_BASE_DIR = '/opt/python/api-helper'

def deploy():
    newdir = 'www-%s' % datetime.now().strftime('%y-%m-%d_%H.%M.%S')
    # 删除已有的tar文件:
    sudo('rm -f %s' % _REMOTE_TMP_TAR)
    # 上传新的tar文件:
    put('dist/%s' % _TAR_FILE, _REMOTE_TMP_TAR)
    # 创建新目录:
 #   with cd(_REMOTE_BASE_DIR):
    sudo('mkdir %s/%s' % (_REMOTE_BASE_DIR,newdir))
    # 解压到新目录:
 #   with cd('%s/%s' % (_REMOTE_BASE_DIR, newdir)):
    sudo('tar -zxvf %s -C %s/%s' % (_REMOTE_TMP_TAR, _REMOTE_BASE_DIR,newdir))
    # 重置软链接:
    with cd(_REMOTE_BASE_DIR):
        sudo('rm -rf www')
        sudo('ln -s %s www' % newdir)
        sudo('chown www:www www')
        sudo('chown -R www:www %s' % newdir)
    # 重启Python服务和nginx服务器:
    with settings(warn_only=True):
      #  sudo('supervisorctl stop apihelper')
      #  sudo('supervisorctl start apihelper')
        sudo('/etc/init.d/nginx reload')

print (lcd(os.path.join(os.path.abspath('.'))))
```
