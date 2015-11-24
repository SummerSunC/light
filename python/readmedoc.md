# 使用readthedoc管理文档

## 资料
[官方文档](https://github.com/rtfd/readthedocs.org/blob/master/docs/getting_started.rst#id5)
[markdown的文档](http://www.mkdocs.org/user-guide/configuration/)
## 使用
### 初始化
```
$ cd docs
$ sphinx-quickstart
```
### 增加md支持
修改conf.py文件
``` python
source_suffix = ['.rst', '.md']
source_parsers = {
    '.md': CommonMarkParser,
}
```

### 准备
将.md或rst文件拷贝到根目录下

### 主题
文件尾部追加
```python
on_rtd = os.environ.get('READTHEDOCS', None) == 'True'

if not on_rtd:  # only import and set the theme if we're building docs locally
    import sphinx_rtd_theme
    html_theme = 'sphinx_rtd_theme'
    html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
```

### 生成
```shell
$ make html
```
