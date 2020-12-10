# 1. CPU相关

```shell
# 查询
top -H -p pid
```



# 2. Load相关

## 2.1 磁盘IO

```shell
iostat -d -x -m 1 10

disk属性值说明：
rrqm/s: 每秒进行 merge 的读操作数目。即 rmerge/s
wrqm/s: 每秒进行 merge 的写操作数目。即 wmerge/s
r/s: 每秒完成的读 I/O 设备次数。即 rio/s
w/s: 每秒完成的写 I/O 设备次数。即 wio/s
rsec/s: 每秒读扇区数。即 rsect/s
wsec/s: 每秒写扇区数。即 wsect/s
rkB/s: 每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。
wkB/s: 每秒写K字节数。是 wsect/s 的一半。
avgrq-sz: 平均每次设备I/O操作的数据大小 (扇区)。
avgqu-sz: 平均I/O队列长度。
await: 平均每次设备I/O操作的等待时间 (毫秒)。
svctm: 平均每次设备I/O操作的服务时间 (毫秒)。
%util: 一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比
```

[[cpu使用率低负载高，原因分析](https://my.oschina.net/JerryBaby/blog/1142980)](https://my.oschina.net/JerryBaby/blog/1142980)



## 2.2 网络IO

[linux系统监控 sar命令详解](https://blog.csdn.net/hguisu/article/details/7493661)

常用命令如下：

```shell
sar -n DEV 1 10

# 每10秒采样一次，连续采样3次，观察CPU 的使用情况；iowait和idle
sar -u -o sys_info  10 3
```



### 2.2.1 socket相关

Linux查看Socket状态：

```shell
# 查看ipv4的状态
> cat /proc/net/sockstat

sockets: used 147777
TCP: inuse 896 orphan 11 tw 139172 alloc 143982 mem 7745
UDP: inuse 14 mem 11
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0

说明：
sockets: used：已使用的所有协议套接字总量
TCP: inuse：正在使用（正在侦听）的TCP套接字数量。其值≤ netstat –lnt | grep ^tcp | wc –l
TCP: orphan：无主（不属于任何进程）的TCP连接数（无用、待销毁的TCP socket数）
TCP: tw：等待关闭的TCP连接数。其值等于netstat –ant | grep TIME_WAIT | wc –l
TCP：alloc(allocated)：已分配（已建立、已申请到sk_buff）的TCP套接字数量。其值等于netstat –ant | grep ^tcp | wc –l
TCP：mem：套接字缓冲区使用量（单位不详(可能是页)。用scp实测，速度在4803.9kB/s时：其值=11，netstat –ant 中相应的22端口的Recv-Q＝0，Send-Q≈400）
UDP：inuse：正在使用的UDP套接字数量
RAW：
FRAG：使用的IP段数量

# IPv6请看：
cat /proc/net/sockstat6
```



# 3. 内存相关

## 3.1 JVM堆dump

```shell
# 打印当前堆栈，详细可查看 jmap -help
jmap -histo pid

# 打印当前堆栈的大小
jmap -heap pid

# 打印文件
jmap  -F -dump:format=b,file=filename pid
```

## 3.2 pmap

pmap 提供了进程的内存映射，pmap 命令用于显示一个或多个进程的内存状态。

```shell
pmap -x pid
### 以下为输出
Address           Kbytes     RSS   Dirty Mode  Mapping
0000000000400000       4       4       0 r-x-- python2.7
### 说明如下
Address: 内存开始地址
Kbytes: 占用内存的字节数（KB）
RSS: 保留内存的字节数（KB）
Dirty: 脏页的字节数（包括共享和私有的）（KB）
Mode: 内存的权限：read、write、execute、shared、private (写时复制)
Mapping: 占用内存的文件、或[anon]（分配的内存）、或[stack]（堆栈）
Offset: 文件偏移
Device: 设备名 (major:minor)
```

