## 利用spark同步Hbase数据到ElasticSearch

***

**主要实现思路:利用spark-submit 命令 + Scala 代码(亲测:同步10万条数据,38s左右)**

具体实现步骤如下:

##### 1.安装hadoop、habse、zooker、sacla、spark

参考此篇博客进行搭建:

[https://blog.csdn.net/qq_51235856/article/details/125712898]: 

##### 2.编写同步jar包(利用scala语言，编写项目，打包成jar，上传到服务器)

* Idea中安装Scala插件

  ![image-20250320172431359](F:\GitDown\hexo\source\_posts\利用spark同步Hbase数据到ElasticSearch.assets\image-20250320172431359.png)

* 新建Scala项目

![image-20250320172304319](F:\GitDown\hexo\source\_posts\利用spark同步Hbase数据到ElasticSearch.assets\image-20250320172304319.png)

* 在build.sbt中引入依赖,导入的依赖需要考虑兼容性

```sbt
ThisBuild / version := "v1"

ThisBuild / scalaVersion := "2.11.12"

lazy val root = (project in file("."))
  .settings(
    name := "spark2es",
    idePackagePrefix := Some("com.xxxx.xxxx")
  )

libraryDependencies += "org.apache.spark" %% "spark-core" % "2.4.0"
libraryDependencies += "org.elasticsearch" %% "elasticsearch-spark-20" % "7.17.25"
libraryDependencies += "org.apache.hbase" % "hbase-client" % "2.5.6"
libraryDependencies += "org.apache.hbase" % "hbase-server" % "2.5.6"
libraryDependencies += "org.apache.hbase.connectors.spark" % "hbase-spark" % "1.0.0"
libraryDependencies += "org.apache.spark" %% "spark-sql" % "2.4.0"
libraryDependencies += "com.alibaba" % "fastjson" % "1.2.77"
libraryDependencies += "org.apache.hbase" % "hbase-mapreduce" % "2.5.6"
libraryDependencies += "org.scala-lang" % "scala-library" % "2.11.12"
```

* 新建scala文件

  ![image-20250320172907896](F:\GitDown\hexo\source\_posts\利用spark同步Hbase数据到ElasticSearch.assets\image-20250320172907896.png)

* 编写文件

```scala
object HBaseToElasticsearchMultipleTables {
  private final val logger: Logger = Logger.getLogger(HBaseToElasticsearchMultipleTables.getClass)
  logger.setLevel(Level.DEBUG)
	
  //定义Hbase中需要同步的表
  private val parserMapping: Map[String, Result => Map[String, String]] = Map(
    "student" -> parsestudent,
	....更多数据
  )

  // 定义多个表和对应的索引
  private final val tablesAndIndexes = Seq(
    //student:Hbase表名 student:es索引名 atsn:列簇
    ("student", "student", "atsn"),
    ....更多数据
  )

  //student
  def parsestudent(result: Result): Map[String, String] = {
    //Hbase中的rowkey
    val rowKey = Bytes.toString(result.getRow)
    val userId = Bytes.toString(result.getValue(Bytes.toBytes("atsn"), Bytes.toBytes("userId")))
	....更多数据

    // 构建 Elasticsearch 文档
    Map(
      //key:Es中的字段名 rowKey:Hbase的列名
      "key" -> rowKey,
      "userId" -> userId,
	  ....更多数据
    )
  }

  def main(args: Array[String]): Unit = {
    // 创建 SparkSession
    val spark = SparkSession.builder()
      .appName("HBaseToElasticsearch")
      //true:连接器将禁用节点发现功能
      .config("spark.es.nodes.wan.only", "true")
      //指定es中id为Hbase中的rowkey,也可以由es自动生成
      .config("spark.es.mapping.id", "key")
      // upsert操作
      .config("spark.es.write.operation", "upsert") 
      // 自动创建索引
      .config("spark.es.index.auto.create", "true") 
      .getOrCreate()
    logger.info("Elasticsearch init Success~~~")

    val startTime = System.currentTimeMillis()
    // HBase 配置
    val hbaseConf = HBaseConfiguration.create()

    // 统计同步的总条数
    var totalCount = 0L;
    var metricsList = List.empty[Map[String, Any]]
    //循环遍历表和索引
    tablesAndIndexes.foreach { case (hbaseTable, esIndex, family) =>
      hbaseConf.set(TableInputFormat.INPUT_TABLE, hbaseTable) // HBase 表名

      // 定义 Scan
      val scan = new Scan()
      scan.addFamily(Bytes.toBytes(family)) // 列族名

      val scanStr = Base64.getEncoder.encodeToString(ProtobufUtil.toScan(scan).toByteArray)
      hbaseConf.set(TableInputFormat.SCAN, scanStr)

      // 读取 HBase 数据
      val hbaseRDD = spark.sparkContext.newAPIHadoopRDD(
        hbaseConf,
        classOf[TableInputFormat],
        classOf[ImmutableBytesWritable],
        classOf[Result]
      )


      // 处理数据并转换为 Elasticsearch 格式
      val parseFunction = parserMapping.getOrElse(hbaseTable,
        throw new IllegalArgumentException(s"未注册的表解析器: $hbaseTable"))
	 //增加 Spark 作业的并行度,提高写入性能
      val targetPartitions = spark.sparkContext.defaultParallelism * 4
      logger.info(s"hbaseRDD.repartition ${targetPartitions}")
	  
      //映射Hbase中的列名和Es中的字段名
      val esRDD = hbaseRDD.repartition(targetPartitions).mapPartitions { partition =>
        partition.flatMap { case (_, result) =>
          try {
            if (result != null) Some(parseFunction(result)) else None
          } catch {
            case e: Exception =>
              // 建议在此添加异常日志记录逻辑
              logger.info(s"Error processing result: ${e.getMessage}")
              None
          }
        }
      }.persist(StorageLevel.MEMORY_ONLY_SER)

      try {
        // 将数据写入 Elasticsearch
        esRDD.saveToEs(s"$esIndex/_doc", Map("es.mapping.id" -> "key"))
        logger.info(s"Elasticsearch Index ${esIndex} sync success~~~")

        // 将该索引的统计指标封装成 Map
        val count = esRDD.count()
        totalCount += count
        val metric = Map("hbaseTable" -> hbaseTable, "count" -> count)

        metricsList = metric :: metricsList

      } catch {
        case e: Exception => {
          logger.error("插入es错误!" + e.getMessage)
          e.printStackTrace()
        }
      } finally {
        esRDD.unpersist()
      }

    }

    val endTime = System.currentTimeMillis()
    val btTime = endTime - startTime
    logger.info("Sync Time:" + btTime + "ms~~~")

    // 将总体执行时间等信息封装成 Map 对象
    val jobName = "HBaseToESSync_" + String.valueOf(System.currentTimeMillis());
    val sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
    val metrics = Map(
      "jobName" -> jobName,
      "totalCount" -> totalCount,
      "jobDetail" -> metricsList,
      "startTime" -> sdf.format(new Date(startTime)),
      "endTime" -> sdf.format(new Date(endTime)),
      "executionTime_ms" -> btTime,
      "timestamp" -> System.currentTimeMillis()
    )

    // 将指标写入到 ES 中的另一个索引（例如 job_metrics）
    spark.sparkContext.parallelize(Seq(metrics))
      .saveToEs("job_metrics/_doc", Map("es.mapping.id" -> "jobName"))

    spark.stop()
  }

}
```

* 打包文件 clean package

![image-20250320173103194](F:\GitDown\hexo\source\_posts\利用spark同步Hbase数据到ElasticSearch.assets\image-20250320173103194.png)



##### 3.编写spark执行脚本

```shell
#!/bin/bash
filename=/root/spark-2.4.0-bin/dist/$(date +%Y)/$(date +%m)/$(date +%d)
if [ ! -d "$filename" ]; then
    mkdir -p "$filename"
fi

/root/spark-2.4.0-bin/bin/spark-submit \
	--class com.xxxx.xxxx.HBaseToElasticsearchMultipleTables \
	--master local[*] \
	--packages org.scala-lang:scala-library:2.11.12,org.apache.hbase:hbase-client:2.5.6,org.apache.hbase:hbase-mapreduce:2.5.6,org.elasticsearch:elasticsearch-spark-20_2.11:7.17.25 \
	--conf spark.es.nodes="xx.xx.xx.xx:9200" \	//Elasticsearch 节点的地址
	--conf spark.es.batch.size.bytes="10mb" \	//控制写入 Elasticsearch 时的批处理大小
	--conf spark.es.batch.size.entries="5000" \	//控制写入 Elasticsearch 时的批处理大小
	--conf spark.es.batch.write.refresh="false" \	//设置为 false，以减少写入时的刷新频率，提高性能
	--conf spark.hbase.zookeeper.quorum="hbase1" \	//配置 HBase 集群信息
	--conf spark.hbase.zookeeper.property.clientPort="2181" \	//配置 HBase 的 Zookeeper 集群信息
	--conf spark.serializer="org.apache.spark.serializer.KryoSerializer" \	//指定使用 Kryo 序列化器，以提高序列化性能
	/root/spark-2.4.0-bin/dist/spark2es_2.11-v1.jar >> $filename/$(date +%Y%m%d)_sync.log 2>&1 &	//指定包含主类的可执行 JAR 文件,将输出日志重定向到指定文件，并在后台运行该命令
```

* class:Jar包执行的主类
* packages:spark执行需要加载的依赖
* conf:spark连接的配置项,在这配置增加了灵活性,也可以在代码中配置
* /root/spark-2.4.0-bin/dist/spark2es_2.11-v1.jar:指定jar路劲,输出执行日志

##### 4.服务器设置定时任务crontab

执行crontab -e

```shel
10 07 * * *  /root/spark-2.4.0-bin/dist/run_sync.sh
```

##### 5.同步阿里云hbase数据，需要添加认证参数

```shell
	kinit -kt /path/to/your.keytab your_user@YOUR-REALM.COM
	#####添加 Kerberos 认证和 HBase 相关配置
	--conf spark.driver.extraJavaOptions="-Djava.security.krb5.conf=/etc/krb5.conf" \
	--conf spark.executor.extraJavaOptions="-Djava.security.krb5.conf=/etc/krb5.conf" \
	--conf spark.hadoop.hbase.security.authentication="kerberos" \
	--conf spark.hadoop.hbase.master.kerberos.principal="hbase/_HOST@YOUR-REALM.COM" \
	--conf spark.hadoop.hbase.regionserver.kerberos.principal="hbase/_HOST@YOUR-REALM.COM" \
```

##### 6.在 spark-submit 中用 --files 传递 hbase-site.xml

```shell
--files /etc/hbase/conf/hbase-site.xml \
--conf spark.executor.extraClassPath=./hbase-site.xml \
--conf spark.driver.extraClassPath=./hbase-site.xml \
#并在代码中加载它
hBaseConf.addResource("hbase-site.xml")
```

##### 7.ElasticSearch可以看到有定时任务的索引生成,可以看到执行的时间以及同步的数据量大小

![image-20250320174118378](F:\GitDown\hexo\source\_posts\利用spark同步Hbase数据到ElasticSearch.assets\image-20250320174118378.png)
