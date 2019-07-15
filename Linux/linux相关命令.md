# 命令记录
查看Apache的并发请求数及其TCP连接状态：

```shell
netstat -n | awk '/^tcp/ {++y[$NF]} END {for(w in y) print w, y[w]}'
```

|状态 | 值  | 备注 
| -- | --- | ---- 
|TIME_WAIT | 1507 |  等待足够的时间以确保远程TCP接收到连接中断请求的确认
| FIN_WAIT1 | 1 | 等待远程TCP连接中断请求，或先前的连接中断请求的确认
FIN_WAIT2  | 1  | 从远程TCP等待连接中断请求
ESTABLISHED  | 55  | 代表一个打开的连接
SYN_RECV  | 21  | 再收到和发送一个连接请求后等待对方对连接请求的确认
CLOSING  | 2  | 没有任何连接状态
LAST_ACK  | 4  | 等待原来的发向远程TCP的连接中断请求的确认

- linux查看端口使用命令：lsof -i
  netstat -tln | grep "8604"
- 每秒上下文切换次数：
  cat /proc/stat | grep ctxt && sleep 30 && cat /proc/stat | grep ctxt
- 统计连接状态：netstat -n | awk '/^tcp/ {++y[$NF]} END {for(w in y) print w, y[w]}'
- 删除${dirName}下{time}外的文件：find ${dirName}  -mtime +${time} -exec rm -rf {} \;
- cat /proc/cpuinfo cpu信息
- 按行输出每一条记录：
  awk -F ':' '{for(i=0;i<NF;i++){printf("%s\n",$i )}}''END{printf("总记录数为：%d\n" , NF)}'
- 删除文件夹下名为xx的文件
  find . -name "xx" | xargs rm -rf 
- 配置错误导致命令不可用/bin 不在PATH 环境变量中，故无法找到该命令
```
最简单的解决方法：
执行此命令语句：
/usr/local$ 
 export PATH="/usr/sbin:/usr/bin:/usr/local/bin:/usr/local/sbin:/bin:/sbin"
或者
export PATH="/usr/sbin:/usr/bin:/usr/local/bin:/usr/local/sbin:/bin:/sbin"
```


# awk经验

1. awk中`print $0`会输出整行数据
2. 内置match函数执行完后允许使用两个变量, `RLENGTH`是匹配的字符串的长度，`RSTART`是匹配的字符串第一次出现的下标

# 磁盘空间

1. [df和du的原理](https://blog.csdn.net/sch0120/article/details/50053295)

