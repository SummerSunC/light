# api文档工具

- mkdocs
- springdocs
- swagger
- [apidocjs](http://apidocjs.com/)

## mkdocs
－ 安装python

```shell
pip3 install mkdocs
# 进入文档目录
mkdocs new {pro}
# 生成站点
mkdocs build
# 本地运行(port : 8080)
mkdocs serve
```

## swagger
- maven 引入swagger-mvc
- 代码中加入@Api*注解
- 将github中swagger-ui,dist目录内容拷贝到本地，发布到服务器

> 跨域问题可利用在header中加入access-control-allow-origin {ip or \*}解决

## apidocjs

- 安装node.js

```shell
npm install apidoc -g

# myapp根目录下要放apidoc.json配置文件
apidoc -i myapp/ -o apidoc/ -t mytemplate/

```
### apidoc.json
``` json
{
  "name": "example",
  "version": "0.1.0",
  "description": "apiDoc basic example",
  "title": "Custom apiDoc browser title",
  "url" : "https://api.github.com/v1"
}
```
