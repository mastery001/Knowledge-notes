[TOC]

# git文件结构

- description:仅供 GitWeb 程序使用，我们无需关心
- config:包含项目特有的配置选项
- info:包含一个全局性排除（global exclude）文件，用以放置那些不希望被记录在 .gitignore 文件中的忽略模式（ignored patterns）
- hooks:包含客户端或服务端的钩子脚本（hook scripts）
- objects:存储所有数据内容
- refs:存储指向数据（分支）的提交对象的指针
- HEAD:指示目前被检出的分支
- index:保存暂存区信息

# 命令
1. git hash-object -w 数据来源(stdin或文件) 
  输出长度为 40 个字符的校验和(一个将待存储的数据外加一个头部信息（header）一起做 SHA-1 校验运算而得的校验和)
2. git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4 
  输出校验值对应的内容，-p是判断其值并显示其内容 , -t为输出该hash的类型，取值有blob,tree,commit
3. git update-index --add --cacheinfo 文件模式 SHA-1 文件名
  必须为上述命令指定 --add 选项，因为此前该文件并不在暂存区中;同样必需的还有 --cacheinfo 选项，因为将要添加的文件位于 Git 数据库中，而不是位于当前目录下。
  文件模式：
    - 100644：普通文件
    - 100755：可执行文件
    - 120000：符号链接
    - 三种模式即是 Git 文件（即数据对象）的所有合法模式（当然，还有其他一些模式，但用于目录项和子模块）。
4. git write-tree:将暂存区内容写入一个树对象
5. git commit-tree
  创建一个提交对象，为此需要指定一个树对象的 SHA-1 值，以及该提交的父提交对象（如果有的话

# pygit
地址：[pygit](https://github.com/benhoyt/pygit)
文档：
1. [500行Python代码实现的Git客户端](http://geek.csdn.net/news/detail/195294)
2. [pygit英文版](http://benhoyt.com/writings/pygit/)
