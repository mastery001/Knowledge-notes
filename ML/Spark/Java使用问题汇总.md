1.java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.sql.metadata.SessionHiveMetaStoreClient

分析：从错误提示上面就知道，spark无法知道hive的元数据的位置，所以就无法实例化对应的client。 解决的办法就是必须将hive-site.xml拷贝到spark/conf目录下

2.Spark not Serializable 使用了非序列化的对象，在Java中若是在类中spark调用使用了匿名函数，则需要将该类实现Serializable接口，并且将成员变量用transient修饰

3.启动spark时加载了hive配置 （1） java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient Caused by: MetaException(message:Version information not found in metastore. ) hive-site.xml 中的 “hive.metastore.schema.verification” 值为 false 即可解决

Caused by: MetaException(message:Could not connect to meta store using any of the URIs provided. Most recent failure: 因为没有正常启动Hive 的 Metastore Server服务进程。 ：nohup hive --service metastore &

（2）org.datanucleus.store.rdbms.connectionpool.DatastoreDriverNotFoundException: The specified datastore driver ("com.mysql.jdbc.Driver") was not found in the CLASSPATH. Please check your CLASSPATH specification, and the name of the driver.

在spark-env.sh文件加入export SPARK_CLASSPATH="/Users/zouziwen/soft/spark-1.6.3/lib/mysql-connector-java-5.0.8-bin.jar"

（3）java.lang.OutOfMemoryError: PermGen space -Xms1024m -Xmx1024m -XX:MaxNewSize=256m -XX:MaxPermSize=256m

（4）java.lang.NoClassDefFoundError: javax/jdo/JDOException 将spark目录下lib的jar包加入到运行classpath中

（5）org.apache.spark.sql.AnalysisException: Table not found idea运行时找不到hive-site.xml，需要将该文件加入到idea的运行环境中

（6）HDFS error: could only be replicated to 0 nodes, instead of 1

```
stop all hadoop services
delete dfs/name and dfs/data directories
hadoop namenode -format # Answer with a capital Y
start hadoop services

```

4.Java对象不能在Spark执行函数中进行更改

5.hive启动问题 hadoop dfsadmin -safemode leave

6.使用map导致程序卡住 由于map的rehash方法不断执行，导致wait

7.key数量不均匀或value数量不均匀，会导致数据倾斜问题，使得数据执行shuffly算子操作时，大量task处于停滞状态 优化指南：<https://tech.meituan.com/spark-tuning-basic.html> <https://tech.meituan.com/spark-tuning-pro.html><http://ganhuo.jo26.cn/doc/d01b9be103e6a.html>