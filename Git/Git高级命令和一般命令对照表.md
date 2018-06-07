| 一般命令              | 高级命令                                                     |
| --------------------- | ------------------------------------------------------------ |
| git add {filename}    | git hash-object -w {filename} 获取文件的hash值 git update-index --add --cacheinfo 100644 {1中的hash值} {1中的filename} 将文件添加至缓冲区 |
| git commit -m ""      | git write-tree 生成tree，并得到该tree的值 echo "" \| git commit-tree {tree的值 } [-p {parent hash value}]或git update-ref refs/heads/master {tree的值 } |
| git branch            | git symbolic-ref HEAD 查看当前分支                           |
| git checkout [branch] | git symbolic-ref HEAD refs/heads/[branch] 切换分支           |