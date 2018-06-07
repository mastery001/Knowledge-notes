# ZooKeeper的数据模型

ZooKeeper有一个类似分布式文件系统的命名体系。区别在于Zookeeper每个一个节点或子节点都可以拥有数据。节点路径是一个由斜线分开的绝对路径，注意没有相对路径。只要满足下面要求的unicode字符都可以作为节点路径：

- 空字符不能出现在路径名
- 不能出现以下字符: \u0001 - \u0019 and \u007F - \u009F
- 以下字符不允许使用: \ud800 -uF8FFF, \uFFF0-uFFFF, \uXFFFE - \uXFFFF (where X is a digit 1 - E), \uF0000 - \uFFFFF
- 字符"."可以作为一个名字的一部分, 但是"."和".."不能单独作为相对路径使用, 以下用法都是无效的: "/a/b/./c"或者"/a/b/../c"
- "zookeeper"为保留字符

# ZNodes

ZooKeeper树结构中的节点被称为znode。各个znode维护着一组用来标记数据和访问权限发生变化的版本号。这些版本号组成的状态结构 具有时间戳。Zookeeper使用版本号和时间戳来验证缓存状态，调整更新。 每次znode中的数据发生变化，znode的版本号增加。例如，每当一个客户端恢复数据时，它就接收这个版本的数据，而当一个客户端提交了更新或删除记 录，它必须同时提供这个znode当前正在发生变化的数据的版本。如果这个版本和目前真实的版本不匹配，则提交无效。 __提示，在分布式程序中，一个字节点可以代表一个通用的主机，服务器，集群中的一员，客户端程序等。但是在Zookeeper中，znode代表数据节 点，Servers代表组成了Zookeeper服务的机器; quorum peers refer to the servers that make up an ensemble; 客户端代表任何使用ZooKeeper服务的主机或程序。 znode作为对程序开发来说最重要的信息，有几个特性需要特别关注下：

## Watches

客户端可以在znode上设置Watch。znode发生的变化会触发watch然后清除watch。当一个watch被触发，Zookeeper给客户端发送一个通知。更多关于watch的内容请查看ZooKeeper Watches一节。

## 数据存取

命名空间中每个znode中的数据读写是原子操作。读操作读取znode中的所有数据位，写操作则替换所有数据。每个节点都有一个访问权限控制表 （ACL）来标记谁可以做什么。 zookeeper不是设计成普通的数据库或大型对象存储的。它是用来管理coordination data。coordination data包括配置文件、状态信息、rendezvous等。这些数据结构的一个共同特点就是相对较小——以千字节为准。Zookeeper的客户端和服务 会检查确保每个znode上的数据小于1M，实际平均数据要远远小于1M。 大规模数据的操作会引发一些潜在的问题并且延长在网络和介质之间传输的时间。如果确实需要大型数据的存储，那么可以采用如NFS或HDFS之类的大型数据 存储系统，亦或是在zookeeper中存储指向存储位置的指针。

## 临时节点（Ephemeral Nodes）

zookeeper还有临时节点的概念，这些节点的生命周期依赖于创建它们的session是否活跃。session结束时节点即被销毁。也由于这种特性，临时节点不允许有子节点。

## 序列节点-命名不唯一

当你创建节点的时候，你会需要zookeeper提供一组单调递增的计数来作为路径结尾。这个计数对父znode是唯一的。用%010d的格式——用0来填充的10位数（计数如此命名是为了简单排序）。例如"0000000001"，注意计数器是有符号整型，超过表示范围会溢出。

# ZooKeeper中的时间

zookeeper有很多记录时间的方式：

- Zxid(ZooKeeper Transaction Id)： zookeeper每次发生改动都会增加zxid，zxid越大，发生的时间越靠后。
- Version numbers： 对znode的改动会增加版本号。版本号包括version (znode上数据的修改数), cversion (znode的子节点的修改数), aversion (znode上ACL（权限）的修改数)。
- Ticks : 多个server构成zookeeper服务时，各个server用ticks来标记如状态上报、连接超时等事件。ticks time还间接反映了session超时的最小值（两次tick time）；如果客户端请求的最小session timeout低于这个最小值，服务端会通知客户端最小超时置为这个最小值。
- Real time : 除了每次znode创建或改动时候将时间戳记录到状态结构中外，zookeeper不使用时钟时间。

# ZooKeeper状态结构(Stat Structure)

存在于znode中的状态结构，由以下各个部分组成：

- czxid - znode创建产生的zxid
- mzxid - znode最后一次修改的zxid
- ctime - znode创建的时间的绝对毫秒数
- mtime - znode最后一次修改的绝对毫秒数
- version - znode上数据的修改数
- cversion - 子节点修改数
- aversion - znode的ACL修改数
- ephemeralOwner - 临时节点的所有者的session id。如果此节点非临时节点，该值为0
- dataLength - znode的数据长度 numChildren - znode子节点数

# ZooKeeper Sessions

客户端通过创建一个handle和服务端建立session连接。一旦创建完成，handle就进入了CONNECTING状态，客户端库尝试连接一台构成zookeeper的server，届时进入CONNECTED状态。通常情况下操作会介于这两种状态之间。 一旦出现了不可恢复的错误：如session中止，鉴权失败或者应用直接结束handle，则handle会进入到CLOSED状态。下图是客户端的状态转换图：

![img](https://ws4.sinaimg.cn/large/006tNc79ly1fn5wj3l8tyj30kl0bsdhn.jpg)

应用在创建客户端session时必须提供一串逗号分隔的主机号:端口号，每对主机端口号对应一个ZooKeeper的 server（如："127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"），客户端库会尝试连接任意一台服务 器，如果连接失败或是客户端主动断开连接，客户端会自动继续与下一台服务器连接，直到连接成功。K

**3.2.0版本新增内容** 一个新的操作“chroot”可以添加在连接字符串的尾部，用来指明客户端命令运行的根目录地址。类似unix的chroot命令，例如： "127.0.0.1:4545/app/a" or "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002/app/a"，说明客户端会以"/app/a"为根目 录，所有路径都相对于根目录来设置，如"/foo/bar"的操作会运行在"/app/a/foo/bar"。 这一特性在多用户环境下非常好用，每个使用zookeeper服务的用户可以设置不同的根目录。 当客户端获得和zookeeper服务连接的handle时，zookeeper会创建一个Zookeeper session分配给客户端，用一个64-bit数字表示。一旦客户端连接了其他服务器，客户端必须把这个session id也作为连接握手的一部分发送。出于安全目的，zookeeper给session id创建一个密码，任何zookeeper服务器都可以验证密码。 当客户端创建session时密码和session id一起发送到客户端来，当客户端重新连接其他服务器时，同时要发送密码和session id。 zookeeper客户端库里有一个创建zookeeper session的参数，叫做session timeout（超时），用毫秒表示。客户端发送请求超时，服务端在超时范围内响应客户端。session超时最小为2个ticktime，最大为20个 ticktime。zookeeper客户端API可以协调超时时间。 当客户端和zookeeper服务器集群断开时，它会搜索session创建时的服务器列表。最后，当至少一个服务器和客户端重新建立连 接，session或被重新置为"connected"状态（超时时间内重新连接），或被置为"expired（过期）"状态（超出超时时间）。不建议在 断开连接后重新创建session。ZK客户端库会帮你重新连接。特别地，我们将启发式学习模式植入客户的库中来处理类似“羊群效应”等问题。只有当你的 session过期时才重新创建（托管的）。 session过期的状态转换图示例同过期session的watcher：

1. 'connected' : session正确创建，客户端和服务集群正常连接
2. .... 客户端从服务器集群断开
3. 'disconnected' : 客户端失去和服务器集群的连接
4. .... 过了一段时间, 超过了集群判定session过期的超时时间, 客户端并没有发觉自己和服务集群断开了连接
5. .... 又过一段时间, 客户端恢复了同集群的网络连接
6. 'expired' : 最终客户端重新连上集群，然后被通知已经到期 另一个session建立时zookeeper需要的参数是默认watcher（监视者）。在客户端发生任何变化时，watcher都会发出通知。 例如客户端失去和服务器的连接、客户端session到期等。watcher默认的初始状态是disconnected。（也就是说任何状态改变事件都由 客户端库发送到watcher）。当新建一个连接时，第一个发送给watcher的事件通常就是session连接事件。 客户端发送请求会使session保持活动状态。客户端会发送ping包（译者注：心跳包）以保持session不会超时。Ping包不仅让服务端 知道客户端仍然活动，而且让客户端也知道和服务端的连接没有中断。Ping包发送时间正好可以判断是否连接中断或是重新启动一个新的服务器连接。 和服务器的连接建立成功，当一个同步或异步操作执行后，有两种情况会让客户端库判断失去连接：
   - 应用在已经失效的session上调用了一个操作时
   - zookeeper服务器有未完成的操作，客户端这时会断开连接。即服务器有未完成的异步调用时 3.2.0版本新增内容 —— SessionMovedException 一个客户端无法查看的内部异常SessionMovedException。这个异常发生在服务端收到一个请求，这个请求的session已经在另一个服 务器上重新连接。发生这种情况的原因通常是客户端发送完请求后，由于网络延时，客户端超时重新和其他服务器建立连接，当请求包到达第一台服务器时，服务器 发现session已经移除并关闭了和客户端的连接。客户端一般不用理会这个问题，但是有一种情况值得注意，当两台客户端使用事先存储的session id和密码试图创建同一个连接时，第一台客户端重建连接，第二台则会被中断。 ZooKeeper Watches

所有zookeeper的读操作——getData(), getChildren(), exists()——都可以设置一个watch。Zookeeper的watch的定义是：watch事件是一次性触发的，发送到客户端的。在监视的数据 发生变化时产生watch事件。以下三点是watch(事件)定义的关键点： 一次性触发: 当数据发生变化时，一个watch事件被发送给客户端。例如，如果一个客户端做了一次getData("/znode1", true)然后节点/znode1发生数据变化或删除，这个客户端将收到/znode1的watch事件。如果/znode1继续发生改变，不会再有watch发送，除非客户端又做了其他读操作产生了新的watch。 发送给客户端: 这就意味着，事件在发往客户端的过程中，可能无法在修改操作成功的返回值到达客户端之前到达客户端。watch是异步发送给watchers的。 zookeeper提供一种保证顺序的方法：客户端在第一次看到某个watch事件之前不可能看到产生watch的修改的返回值。网络延时或其他因素可能 导致不同客户端看到watch并返回不同时间更新的返回值。关键的一点是，不同的客户端看到发生的一切都必须是按照相同顺序的。 watch依附的数据: 这是说改变一个节点有不通方式。用好理解的话说，zookeeper维护两组watch：data watch和child watch。getData()和exists()产生data watch。getChildren()引起child watch。watch根据数据返回的种类不同而不同。getData()和exists()返回关于节点的数据信息，而getChildren()返回 子节点列表。因此setData()触发某个znode的data watch（假设事件成功）。create()成功会触发被创建的znode上的data watch和在它父节点上的child watch。delete()成功会触发data watch和child watch（因为没有了子节点）。 watch在客户端已连接上的服务器里维护，这样可以保证watch轻量便于设置，维护和分发。当客户端连接了一台新的服务器，watch会在任何 session事件时触发。当断开和服务器的连接时，watch不会触发。当客户端重新连接上时，任何之前注册过的watch都会重新注册并在需要的时候 被触发。一般来说这一切都是透明的。只有一种可能会丢失watch：当一个znode在断开和服务器连接时被创建或删除，那么判断这个znode存在的 watch因未创建而找不到。 ZooKeeper如何保证watch可靠性 zookeeper有如下方式： watch与其他事件、watch、异步回复保持有序，Zookeeper客户端库确保任何分发都是有序的。 客户端会在某个监视的znode数据更新之前看到这个znode的watch事件。 watch事件的顺序由Zookeeper服务端观察到的更新顺序决定。 watch注意事项 watch是一次性触发的；如果你收到watch事件后还想继续得到后续更改的通知，你需要再生成（设置）一个watch。 由于watch是一次性触发，你在获取某事件和发送新的请求来得到watch这个操作之间，无法确保观察到Zookeeper中那个节点在这期间 的所有修改。你要准备好应付这种情况出现：znode会在收到事件和再次设置新事件（译者注：对节点的操作）之间发生了多次修改。（你可能并不关心，但是 必须了解这可能发生） watch对象，或是function/context对，只会在得到通知时触发一次。例如，如果一个watch对象同时用来监控某个目标文件是否存在和监听getData()，之后那个文件被删除了。那么这个watch对象只会触发一次文件删除事件通知。 如果你断开了同服务器的连接（例如服务器挂了），你在重新连上之前得不到任何watch。出于这种原因，session event会被发送给所有重要的watch handler。可以使用session事件进入安全模式：当断开连接时你收不到任何事件，这样你的进程可以在那种模式下稳健地执行。(译者注：可以通过 发送session event使客户端进入安全模式（伪断开连接状态），在安全模式你可以修改代码而不用担心程序收到事件通知) 使用ACL控制ZooKeeper访问权限

zookeeper使用ACL来控制对znode（zookeeper的数据节点）的访问权限。ACL的实现方式和unix的文件权限类似：用不同 位来代表不同的操作限制和组限制。与标准unix权限不同的是，zookeeper的节点没有三种域——用户，组，其他。zookeeper里没有节点的 所有者的概念。取而代之的是，一个由ACL指定的id集合和其相关联的权限。 注意，一个ACL只从属于一个特定的znode。对这个znode子节点也是无效的。例如，如果/app只有被ip172.16.16.1的读权限，/app/status有被所有人读的权限，那么/app/status可以被所有人读，ACL权限不具有递归性。 zookeeper支持插件式认证方式，id使用scheme:id的形式。scheme是id对应的类型方式，例如ip:172.16.16.1就是一个地址为172.16.16.1的主机id。 当客户端连接zookeeper并且认证自己，zookeeper就在这个与客户端的连接中关联所有与客户端一致的id。当客户端访问某个znode时，znode的ACL会重新检查这些id。ACL的表达式为(scheme:expression,perms)。expression就是特殊的scheme，例如，(ip:19.22.0.0/16, READ)就是把任何以19.22开头的ip地址的客户端赋予读权限。 ACL权限

ZooKeeper支持下列权限： CREATE：允许创建子节点 READ：允许获得节点数据并列出所有子节点 WRITE：允许设置节点上的数据 DELETE：允许删除子节点 ADMIN：允许设置权限 CREATE和DELETE操作是更细的粒度上的WRITE操作。有一种特殊的情况： 你想要A获得操作zookeeper上某个znode的权限，但是不可以对其子节点进行CREATE和DELETE。 只CREATE不DELETE：某个客户端在上一级目录上通过发送创建请求创建了一个zookeeper节点。你希望所有客户端都可以在这个节点上添加，但是只有创建者可以删除。（这就类似于文件的APPEND权限） zookeeper没有文件所有者的概念，但有ADMIN权限。在某种意义上说，ADMIN权限指定了所谓的所有者。zookeeper虽然不支持 查找权限（在目录上的执行权限虽然不能列出目录内容，却可以查找），但每个客户端都隐含着拥有查找权限。这样你可以查看节点状态，但仅此而已。（这有个问 题，如果你在不存在的节点上调用了zoo_exists()，你将无权查看） 内建ACL模式

ZooKeeper有下列内建模式： world 有独立id，anyone，代表任何用户。 auth 不使用任何id，代表任何已经认证过的用户 digest 之前使用了格式为username:pathasowrd的字符串来生成一个MD5哈希表作为ACL ID标识。在空文档中发送username:password来完成认证。现在的ACL表达式格式为username:base64, 用SHA1编码密码。 ip 用客户端的ip作为ACL ID标识。ACL表达式的格式为addr/bits，addr中最有效的位匹配上主机ip最有效的位。 Zookeeper的Leader选举： 在QuorumPeer的startLeaderElection方法里包含leader选举的逻辑。Zookeeper默认提供了4种选举方式，默认是第4种: FastLeaderElection。 我们先假设我们这是一个崭新的集群，崭新的集群的选举和之前运行过一段时间的选举是有稍许不同的，后面会提及。 节点状态： 每个集群中的节点都有一个状态 LOOKING, FOLLOWING, LEADING, OBSERVING。都属于这4种，每个节点启动的时候都是LOOKING状态，如果这个节点参与选举但最后不是leader，则状态是FOLLOWING，如果不参与选举则是OBSERVING，leader的状态是LEADING。 开始这个选举算法前，每个节点都会在zoo.cfg上指定的监听端口启动监听(server.1=127.0.0.1:20881:20882)，这里的20882就是这里用于选举的端口。 在FastLeaderElection里有一个Manager的内部类，这个类里有启动了两个线程：WorkerReceiver， WorkerSender。为什么说选举这部分复杂呢，我觉得就是这些线程就像左右互搏一样，非常难以理解。顾名思义，这两个线程一个是处理从别的节点接收消息的，一个是向外发送消息的。对于外面的逻辑接收和发送的逻辑都是异步的。 这里配置好了，QuorumPeer的run方法就开始执行了，这里实现的是一个简单的状态机。因为现在是LOOKING状态，所以进入LOOKING的分支，调用选举算法开始选举了： setCurrentVote(makeLEStrategy().lookForLeader()); 而在lookForLeader里主要是干什么呢？首先我们会更新一下一个叫逻辑时钟的东西，这也是在分布式算法里很重要的一个概念，但是在这里先不介绍，可以参考后面的论文。然后决定我要投票给谁。不过zookeeper这里的选举真直白，每个节点都选自己(汗),选我，选我，选我...... 然后向其他节点广播这个选举信息。这里实际上并没有真正的发送出去，只是将选举信息放到由WorkerSender管理的一个队列里。 synchronized(this){ //逻辑时钟
logicalclock++; //getInitLastLoggedZxid(), getPeerEpoch()这里先不关心是什么，后面会讨论 updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch()); }

//getInitId() 即是获取选谁，id就是myid里指定的那个数字，所以说一定要唯一 private long getInitId(){ if(self.getQuorumVerifier().getVotingMembers().containsKey(self.getId()))
return self.getId(); else return Long.MIN_VALUE; }

//发送选举信息，异步发送 sendNotifications(); 现在我们去看看怎么把投票信息投递出去。这个逻辑在WorkerSender里，WorkerSender从sendqueue里取出投票，然后交给QuorumCnxManager发送。因为前面发送投票信息的时候是向集群所有节点发送，所以当然也包括自己这个节点，所以QuorumCnxManager的发送逻辑里会判断，如果这个要发送的投票信息是发送给自己的，则不发送了，直接进入接收队列。 public void toSend(Long sid, ByteBuffer b) { if (self.getId() == sid) { b.position(0); addToRecvQueue(new Message(b.duplicate(), sid)); } else { //发送给别的节点，判断之前是不是发送过 if (!queueSendMap.containsKey(sid)) { //这个SEND_CAPACITY的大小是1，所以如果之前已经有一个还在等待发送，则会把之前的一个删除掉，发送新的 ArrayBlockingQueue<ByteBuffer> bq = new ArrayBlockingQueue<ByteBuffer>(SEND_CAPACITY); queueSendMap.put(sid, bq); addToSendQueue(bq, b);

```
} else {
             ArrayBlockingQueue<ByteBuffer> bq = queueSendMap.get(sid);
             if(bq != null){
                 addToSendQueue(bq, b);
             } else {
                 LOG.error("No queue for server " + sid);
             }
         }
         //这里是真正的发送逻辑了
         connectOne(sid);
            
    }
}
```

connectOne就是真正发送了。在发送之前会先把自己的id和选举地址发送过去。然后判断要发送节点的id是不是比自己的id大，如果大则不发送了。如果要发送又是启动两个线程：SendWorker,RecvWorker(这种一个进程内许多不同种类的线程，各自干活的状态真的很难理解)。发送逻辑还算简单，就是从刚才放到那个queueSendMap里取出，然后发送。并且发送的时候将发送出去的东西放到一个lastMessageSent的map里，如果queueSendMap里是空的，就发送lastMessageSent里的东西，确保对方一定收到了。 看完了SendWorker的逻辑，再来看看数据接收的逻辑吧。还记得前面提到的有个Listener在选举端口上启动了监听么，现在这里应该接收到数据了。我们可以看到receiveConnection方法。在这里，如果接收到的的信息里的id比自身的id小，则断开连接，并尝试发送消息给这个id对应的节点(当然，如果已经有SendWorker在往这个节点发送数据，则不用了)。 如果接收到的消息的id比当前的大，则会有RecvWorker接收数据，RecvWorker会将接收到的数据放到recvQueue里。 而FastLeaderElection的WorkerReceiver线程里会不断地从这个recvQueue里读取Message处理。在WorkerReceiver会处理一些协议上的事情，比如消息格式等。除此之外还会看看接收到的消息是不是来自投票成员。如果是投票成员，则会看看这个消息里的状态，如果是LOOKING状态并且当前的逻辑时钟比投票消息里的逻辑时钟要高，则会发个通知过去，告诉谁是leader。在这里，刚刚启动的崭新集群，所以逻辑时钟基本上都是相同的，所以这里还没判断出谁是leader。不过在这里我们注意到如果当前节点的状态是LOOKING的话，接收逻辑会将接收到的消息放到FastLeaderElection的recvqueue里。而在FastLeaderElection会从这个recvqueue里读取东西。 这里就是选举的主要逻辑了：totalOrderPredicate

protected boolean totalOrderPredicate(long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {return ((newEpoch > curEpoch) || ((newEpoch == curEpoch) && ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId))))); }

1. 判断消息里的epoch是不是比当前的大，如果大则消息里id对应的server我就承认它是leader
2. 如果epoch相等则判断zxid，如果消息里的zxid比我的大我就承认它是leader
3. 如果前面两个都相等那就比较一下server id吧，如果比我的大我就承认它是leader。 关于前面两个东西暂时我们不去关心它，对于新启动的集群这两者都是相等的。 那这样看来server id的大小也是leader选举的一环啊（有的人生下来注定就不平凡，这都是命啊）。 最后我们来看看，很多文章所介绍的，如果超过一半的人说它是leader，那它就是leader的逻辑吧

private boolean termPredicate( HashMap<Long, Vote> votes, Vote vote) {

```
HashSet<Long> set = new HashSet<Long>();
    //遍历已经收到的投票集合，将等于当前投票的集合取出放到set中
    for (Map.Entry<Long,Vote> entry : votes.entrySet()) {
        if (self.getQuorumVerifier().getVotingMembers().containsKey(entry.getKey())
                && vote.equals(entry.getValue())){
            set.add(entry.getKey());
        }
    }
    
    //统计set，也就是投某个id的票数是否超过一半
    return self.getQuorumVerifier().containsQuorum(set);
}

public boolean containsQuorum(Set<Long> ackSet) {
    return (ackSet.size() > half);
}
```

最后一关：如果选的是自己，则将自己的状态更新为LEADING，否则根据type，要么是FOLLOWING，要么是OBSERVING。 到这里选举就结束了。 这里介绍的是一个新集群启动时候的选举过程，启动的时候就是根据zoo.cfg里的配置，向各个节点广播投票，一般都是选投自己。然后收到投票后就会进行进行判断。如果某个节点收到的投票数超过一半，那么它就是leader了。 了解了这个过程，我们来看看另外一个问题： 一个集群有3台机器，挂了一台后的影响是什么？挂了两台呢？ 挂了一台：挂了一台后就是收不到其中一台的投票，但是有两台可以参与投票，按照上面的逻辑，它们开始都投给自己，后来按照选举的原则，两个人都投票给其中一个，那么就有一个节点获得的票等于2，2 > (3/2)=1 的，超过了半数，这个时候是能选出leader的。 挂了两台： 挂了两台后，怎么弄也只能获得一张票， 1 不大于 (3/2)=1的，这样就无法选出一个leader了。

在前面介绍时，为了简单我假设的是这是一个崭新的刚启动的集群，这样的集群与工作一段时间后的集群有什么不同呢？不同的就是epoch和zxid这两个参数。在新启动的集群里这两个一般是相等的，而工作一段时间后这两个参数有可能有的节点落后其他节点？

zookeeper学习：<http://www.cnblogs.com/yuyijq/p/3424473.html>