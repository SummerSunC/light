# Git使用技巧
[官方Git教程](http://www.git-scm.com/book/zh/v1/)

## git 特点
- git 是分布式的，所有的终端都保存有完整的历史信息
- git 处理很快，因为本地存有想要的所有内容，所以每次差异比较时只要读取本地就好，而不要每次都联网
- Git 不保存这些前后变化的差异数据，只关心整体变化。实际上，Git更像是把变化的文件作快照后，记录在一个微型的文件系统中。

> ![](http://www.git-scm.com/figures/18333fig0105-tn.png)

- git能时刻保存数据记录的完整性，在保存到git之前，所有数据都要进行内容的校验和（checksum）计算（Git 使用SHA-1算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值 e.g:24b9da6552252987aa493b52f8696cd6d3b00373）,这个哈希作为索引
- git中文件三种状态：已提交（committed），已修改（modified）和已暂存（staged）
> ![](http://www.git-scm.com/figures/18333fig0106-tn.png)

