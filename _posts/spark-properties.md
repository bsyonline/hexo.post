---
title: Spark Properties
tags:
- Spark
category:
- Spark
author: bsyonline
lede: 没有摘要
date: 2017-12-26 15:16:56
thumbnail:
---

### Application Properties
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.app.name</code></td><td>(none)</td><td> 应用程序名称，会在 UI 和日志中显示。</td></tr></tr><tr><td><code>spark.driver.cores</code></td><td>1</td><td>驱动程序使用的核心数量，只在集群模式下有效。</td></tr><tr><td><code>spark.driver.maxResultSize</code></td><td>1g</td><td>限制每个 Spark action 所有分区序列化结果的大小。最小设置为 1 M，0 为不限制。如果超出限制，作业会终止。该值设置过大，会导致内存溢出。</td></tr><tr><td><code>spark.driver.memory</code></td><td>1g</td><td>SparkContext 初始化时驱动程序使用的内存总和 (e.g.<code>1g</code>, <code>2g</code>)。注意： 在 client 模式下，该设置不能通过配置 <code>SparkConf</code> 生效，因为启动程序的 JVM 已经启动了。可以通过<code>--driver-memory</code>命令行选项或在默认配置文件中设置。</td></tr><tr><td><code>spark.executor.memory</code></td><td>1g</td><td>每个 executor 使用的内存总和。 (e.g. <code>2g</code>, <code>8g</code>)</td></tr><tr><td><code>spark.extraListeners</code></td><td>(none)</td><td>可以设置多个实现<code>SparkListener</code>接口的类，用逗号分隔。SparkContext 初始化时，这些类的实例会被创建并注册到 Spark 的监听总线。创建时会调用只接收 SparkConf 为参数的构造函数或无参数的构造函数，如果没有合适的构造函数，SparkContext 会创建失败并抛出异常。</td></tr><tr><td><code>spark.local.dir</code></td><td>/tmp</td><td>Spark 的  "scratch" 空间路径, 用于存放 map 的输出文件和数据集。这个路径应该是一个系统本地路径，也可以是多个磁盘上的路径。	注意：从 Spark 1.0 开始，该参数会被  SPARK_LOCAL_DIRS (Standalone, Mesos)  或 LOCAL_DIRS (YARN) 覆盖。</td></tr><tr><td><code>spark.logConf</code></td><td>false</td><td>SparkContext 启动时，记录有效的 SparkConf 的 INFO 信息。</td></tr><tr><td><code>spark.master</code></td><td>(none)</td><td>集群管理节点地址。<a href="https://spark.apache.org/docs/2.1.1/submitting-applications.html#master-urls">allowed master URL's</a></td></tr><tr><td><code>spark.submit.deployMode</code></td><td>(none)</td><td>Spark 驱动程序的部署模式。"client" 驱动程序在本地启动，"cluster" 启动程序在集群中的某个节点上启动。</td></tr></table>
### Runtime Environment
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.driver.extraClassPath</code></td><td>(none)</td><td>启动程序 classpath 的预加载扩展 classpath 路径。<br>注意：在 client 模式下，不能通过<code>SparkConf</code>设置，应使用<code>--driver-class-path</code>命令行选项或默认配置文件设置。</td></tr><tr><td><code>spark.driver.extraJavaOptions</code></td><td>(none)</td><td>额外的 JVM 选项，比如 GC 或日志。<br>注意：最大堆大内存（-Xmx） 不能通过该项设置。集群模式下，最大堆内存使用<code>spark.driver.memory</code>设置，client 模式下使用<code>--driver-memory</code>命令行选项设置。<br/>注意：在 client 模式下，该项不能在<code>SparkConf</code>设置，应通过<code>--driver-java-options</code>命令行选项或默认配置文件设置。</td></tr><tr><td><code>spark.driver.extraLibraryPath</code></td><td>(none)</td><td>启动驱动程序 JVM 时设置一个特殊的库路径。<br/>注意：在 client 模式下，该项不能在<code>SparkConf</code>设置，应通过<code>--driver-library-path</code>命令行选项或默认配置文件设置。</td></tr><tr><td><code>spark.driver.userClassPathFirst</code></td><td>false</td><td>（实验）是否设置用户的 jar 优先于 Spark 的 jar 。该特性用于解决 Spark 依赖和用户依赖之间的冲突。这个特性还在实验中，只在集群模式下生效。</td></tr><tr><td><code>spark.executor.extraClassPath</code></td><td>(none)</td><td>预设额外的类路径条目，这个属性主要是为了先后兼容老的 Spark 版本，用户通常不需要设置。</td></tr><tr><td><code>spark.executor.extraJavaOptions</code></td><td>(none)</td><td>向 executor 传递一个额外的 JVM 选项，比如 GC 或日志。<br>注意：该项不能设置 Spark Properties 或最大堆大小（-Xmx）。Spark properties 应使用 SparkConf 对象或 spark-defaults.conf 来设置。最大堆内存应通过 <code> spark.executor.memory</code>设置。</td></tr><tr><td><code>spark.executor.extraLibraryPath</code></td><td>(none)</td><td>executor JVM 启动时设置一个特殊库路径。</td></tr><tr><td><code>spark.executor.logs.rolling.maxRetainedFiles</code></td><td>(none)</td><td>设置系统保存的最新日志滚动数量，老的日志文件将被删除。默认禁用。</td></tr><tr><td><code>spark.executor.logs.rolling.enableCompression</code></td><td>false</td><td>启用日志压缩。如果启动，滚动的日志将被压缩。默认禁用。</td></tr><tr><td><code>spark.executor.logs.rolling.maxSize</code></td><td>(none)</td><td>设置日志文件滚动的字节数组最大长度。默认禁用。参考 <code>spark.executor.logs.rolling.maxRetainedFiles</code> 清除旧日志文件。</td></tr><tr><td><code>spark.executor.logs.rolling.strategy</code></td><td>(none)</td><td>设置日志滚动策略。默认禁用，可以基于“时间”滚动或按照“大小”滚动。基于“时间”，使用<code>spark.executor.logs.rolling.time.interval</code>设置滚动频率。按照“大小”，使用<code>spark.executor.logs.rolling.maxSize</code>设置最大滚动文件大小。</td></tr><tr><td><code>spark.executor.logs.rolling.time.interval</code></td><td>daily</td><td>设置日志滚动的时间间隔。默认不滚动。可以设置的值包括<code>daily</code>, <code>hourly</code>, <code>minutely</code> 或任意秒数。参考<code>spark.executor.logs.rolling.maxRetainedFiles</code>自动清除旧的日志。</td></tr><tr><td><code>spark.executor.userClassPathFirst</code></td><td>false</td><td>（实验）和<code>spark.driver.userClassPathFirst</code>功能相同，但是是作用在 executor 实例。</td></tr><tr><td><code>spark.executorEnv.[EnvironmentVariableName]</code></td><td>(none)</td><td>通过指定<code>EnvironmentVariableName</code>添加 executor 的环境变量，可以设置多个值。</td></tr><tr><td><code>spark.python.profile</code></td><td>false</td><td>在 python 的 worker 节点上启动 profile 。profile 的结果通过<code>sc.show_profiles()</code>显示或在启动程序退出前显示。通过<code>sc.dump_profiles(path)</code>设置磁盘的路径。如果 profile 结果被手动隐藏，那么在驱动程序退出时不显示。默认<code>pyspark.profiler.BasicProfiler</code> 启用，但是可以给 SparkContext 构造函数传一个 profiler 类参数覆盖该值。</td></tr><tr><td><code>spark.files</code></td><td></td><td>每个 executor 的工作目录列表，用逗号分隔。</td></tr><tr><td><code>spark.jars</code></td><td></td><td>设置驱动程序和 executor 类路径包含的本地 jar，用逗号分隔。 </td></tr><tr><td><code>spark.jars.packages</code></td><td></td><td>驱动程序和 executor 类路径包含的 jar 包的 maven 坐标。按照本地 maven 仓库，maven 中心库和<code>spark.jars.ivy</code>指定的附加远程仓库顺序查找。格式为：groupId:artifactId:version</td></tr><tr><td><code>spark.jars.excludes</code></td><td></td><td>排除<code>spark.jars.packages</code>提供的 jar 以避免冲突。</td></tr><tr><td><code>spark.jars.ivy</code></td><td></td><td>查找附加远程仓库<code>spark.jars.packages</code>指定的坐标，用逗号分隔。</td></tr></table>
### Shuffle Behavior
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.reducer.maxSizeInFlight</code></td><td>48m</td><td>同时从每个 reduce 任务获取的 map 最大值。每个输出都需要创建缓冲区来接收，所以每个 reduce 任务都有固定的内存开销，因此应设置一个较小的值。</td></tr><tr><td><code>spark.reducer.maxReqsInFlight</code></td><td>Int.MaxValue</td><td>该选项限制任意给定的点接收远程请求的数量。但集群主机的数量增加，可能会有大量的请求连接到一个或多个节点，导致 worker 节点负载过大而出错。限制请求的数量可以避免出现此种问题。</td></tr><tr><td><code>spark.shuffle.compress</code></td><td>true</td><td>是否压缩 map 的输出文件。一般来说使用压缩是的好办法。压缩将使用<code>spark.io.compression.codec</code>属性。</td></tr><tr><td><code>spark.shuffle.file.buffer</code></td><td>32k</td><td>每个 shuffle 文件流的缓冲区大小。缓冲区会降低创建中间 shuffle 文件时的磁盘寻址和系统调用次数。</td></tr><tr><td><code>spark.shuffle.io.maxRetries</code></td><td>3</td><td>（Netty only）抓取到 IO 相关异常错误自动重试。重试使 shuffle 在遇到长时间 GC 暂停或短时间的网络异常更稳定。</td></tr><tr><td><code>spark.shuffle.io.numConnectionsPerPeer</code></td><td>1</td><td>（Netty only）在大规模集群中为了减少连接创建，主机之间的连接被重用。对于集群中有大量的硬盘少量的主机导致并发不足时，用户可以调大该值。</td></tr><tr><td><code>spark.shuffle.io.preferDirectBufs</code></td><td>true</td><td>（Netty only）在 shuffle 和缓存交换过程中，使用堆缓冲区来减少 GC 。对于堆缓冲区内存有限的环境，用户可能希望关闭这个功能，使所有分配都从 Netty 移到堆中。</td></tr><tr><td><code>spark.shuffle.io.retryWait</code></td><td>5s</td><td>（Netty only）每次抓取重试等待时间。默认最大延迟 15 秒<code>maxRetries * retryWait</code>。</td></tr><tr><td><code>spark.shuffle.service.enabled</code></td><td>false</td><td>应用额外的 shuffle 服务。该服务保存 executor 生成的 shuffle 文件，以保证 executor 被安全删除。如果<code>spark.dynamicAllocation.enabled</code>为 true，则该值必须设置为 true 。</td></tr><tr><td><code>spark.shuffle.service.port</code></td><td>7337</td><td>运行额外 shuffle 服务的端口号。</td></tr><tr><td><code>spark.shuffle.service.index.cache.entries</code></td><td>1024</td><td>shuffle 服务的缓存索引中保存的最大条目数量。</td></tr><tr><td><code>spark.shuffle.sort.bypassMergeThreshold</code></td><td>200</td><td>（高级）在基于排序的 shuffle 管理器中，如果没有 map 端的聚合，应避免合并排序数据并减少分区数量。</td></tr><tr><td><code>spark.shuffle.spill.compress</code></td><td>true</td><td>是否压缩 shuffle 过程中的溢出数据。压缩使用<code>spark.io.compression.codec</code>属性。</td></tr><tr><td><code>spark.io.encryption.enabled</code></td><td>false</td><td>使用 IO 加密。除 Mesos 外，其他模式都支持。使用该特性使建议使用 RPC 加密。</td></tr><tr><td><code>spark.io.encryption.keySizeBits</code></td><td>128</td><td>IO 加密的密钥大小。128，192，256 可选。</td></tr><tr><td><code>spark.io.encryption.keygen.algorithm</code></td><td>HmacSHA1</td><td>生成 IO 加密密钥的算法。支持的算法见Java密码体系结构标准算法名称文档的 KeyGenerator 部分。</td></tr></table>
### Spark UI
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.eventLog.compress</code></td><td>false</td><td>如果<code>spark.eventLog.enabled</code>为 true ，是否使用压缩。</td></tr><tr><td><code>spark.eventLog.dir</code></td><td>file:///tmp/spark-events</td><td>如果<code>spark.eventLog.enabled</code>为 true ，设置路径为 Spark 事件日志的根目录。在这个目录下，Spark 为每个应用程序创建一个子路径，记录特殊的事件。用户可以将其设置为一个统一的路径，如 HDFS 上的路径，以便历史可读。</td></tr><tr><td><code>spark.eventLog.enabled</code></td><td>false</td><td>是否记录 Spark 事件日志，程序结束后重现 Web UI 很有用。</td></tr><tr><td><code>spark.ui.enabled</code></td><td>true</td><td>是否运行 web UI 程序。</td></tr><tr><td><code>spark.ui.killEnabled</code></td><td>true</td><td>允许通过 UI 杀掉任务。</td></tr><tr><td><code>spark.ui.port</code></td><td>4040</td><td>应用程序面板端口号。</td></tr><tr><td><code>spark.ui.retainedJobs</code></td><td>1000</td><td>GC 之前 Spark UI 和状态 api 能记录多少作业。</td></tr><tr><td><code>spark.ui.retainedStages</code></td><td>1000</td><td>GC 之前 Spark UI 和状态 api 能记录多少阶段。</td></tr><tr><td><code>spark.ui.retainedTasks</code></td><td>100000</td><td>GC 之前 Spark UI 和状态 api 能记录多少任务。</td></tr><tr><td><code>spark.ui.reverseProxy</code></td><td>false</td><td> Spark 主节点作为 worker 节点和 UI 的反向代理。Spark 主节点不需要访问主机就可以反向代理 worker 节点和 UI 。需要注意的是，worker 节点和 UI 不需要访问路径，只能通过主节点或代理访问。该设置影像所有的 wroker 节点和 UI ，必须在所有的 worker 节点，驱动程序和主节点上设置。</td></tr><tr><td><code>spark.ui.reverseProxyUrl</code></td><td></td><td>代理的 URL 。应包括 （http/https）和端口。</td></tr><tr><td><code>spark.worker.ui.retainedExecutors</code></td><td>1000</td><td>GC 之前 Spark UI 和状态 api 能记录多少完成的 executor 。</td></tr><tr><td><code>spark.worker.ui.retainedDrivers</code></td><td>1000</td><td>GC 之前 Spark UI 和状态 api 能记录多少完成的驱动程序。</td></tr><tr><td><code>spark.sql.ui.retainedExecutions</code></td><td>1000</td><td>GC 之前 Spark UI 和状态 api 能记录多少完成的 execution 。</td></tr><tr><td><code>spark.streaming.ui.retainedBatches</code></td><td>1000</td><td>GC 之前 Spark UI 和状态 api 能记录多少完成的 batch 。</td></tr><tr><td><code>spark.ui.retainedDeadExecutors</code></td><td>100</td><td>GC 之前 Spark UI 和状态 api 能记录多少死亡的 executor 。</td></tr></table>
### Compression and Serialization
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.broadcast.compress</code></td><td>true</td><td>是否在发送广播变量之前压缩</td></tr><tr><td><code>spark.io.compression.codec</code></td><td>lz4</td><td>用来压缩 RDD 分区、广播变量、shuffle 输出等内部数据的解码器。Spark 默认提供三种解码器：<code>lz4</code>, <code>lzf</code>, <code>snappy</code>。也可以使用完全类名来指定解码器，比如<code>org.apache.spark.io.LZ4CompressionCodec</code>,<code>org.apache.spark.io.LZFCompressionCodec</code>, <code>org.apache.spark.io.SnappyCompressionCodec</code>。</td></tr><tr><td><code>spark.io.compression.lz4.blockSize</code></td><td>32k</td><td>LZ4 压缩的块大小。块大小越小，shuffle 使用的内存越小。</td></tr><tr><td><code>spark.io.compression.snappy.blockSize</code></td><td>32k</td><td>Snappy 压缩的块大小。块大小越小，shuffle 使用的内存越小。</td></tr><tr><td><code>spark.kryo.classesToRegister</code></td><td>(none)</td><td>通过逗号分隔列表指定在 Kryo 中注册的类的名字。</td></tr><tr><td><code>spark.kryo.referenceTracking</code></td><td>true</td><td>在使用 Kryo 序列化数据时，是否跟踪相同对象的引用。如果对象图包含循环的话，这是必要的。如果没有，可以禁用来提高性能。</td></tr><tr><td><code>spark.kryo.registrationRequired</code></td><td>false</td><td>是否需要在 Kryo 注册。如果设置为 'true'，未注册的类在序列化时，Kryo 会抛出异常。如果设置为 'false' ，Kryo 会将未注册的类名写到每一个对象，这个会造成巨大的性能开销，所以应保证序列化的类在 Kryo 中注册。</td></tr><tr><td><code>spark.kryo.registrator</code></td><td>(none)</td><td>通过逗号分隔的列表指定在 Kryo 中注册的类名。这在通过自定义方式注册类时很有用，否则使用<code>spark.kryo.classesToRegister</code>更简单。</td></tr><tr><td><code>spark.kryo.unsafe</code></td><td>false</td><td>是否使用不安全的 Kryo 序列化器。使用不安全的 IO 能极大提高性能。</td></tr><tr><td><code>spark.kryoserializer.buffer.max</code></td><td>64m</td><td>Kryo 序列化缓冲区允许的最大值。这个大小应大于任何想序列化的对象的大小。如果出现 "buffer limit exceeded" 异常，则调大该值。</td></tr><tr><td><code>spark.kryoserializer.buffer</code></td><td>64k</td><td>Kryo 序列化缓冲区的初始大小。注意：每个 worker 节点的每一个 core 都有一个缓冲区。<code>spark.kryoserializer.buffer.max</code>可以修改缓冲区的大小。</td></tr><tr><td><code>spark.rdd.compress</code></td><td>false</td><td>是否压缩 RDD 分区。可以节约大量的空间和额外的 CPU 时间。</td></tr><tr><td><code>spark.serializer</code></td><td>org.apache.spark.serializer.JavaSerializer</td><td>用来序列化对象的序列化器。默认的 Java 序列化器可以用于任何 Java 序列化对象，但是性能不高。对性能有要求时推荐使用<code>org.apache.spark.serializer.KryoSerializer</code>。</td></tr><tr><td><code>spark.serializer.objectStreamReset</code></td><td>100</td><td>使用 <code>org.apache.spark.serializer.JavaSerializer</code>时, 序列化器缓存对象会防止多余数据写入会导致这些对象不会被 GC 。通过调用 'reset' 可以刷新序列化器信息，允许旧对象被回收。 默认情况下，每序列化 100 个对象会刷新一次，设置 -1  可以关闭刷新。</td></tr></table>
### Memory Management
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.memory.fraction</code></td><td>0.6</td><td>执行和存储的内存比例（堆内存-300M）。比例越小，缓存数据回收和泄露越频繁。建议使用默认值。</td></tr><tr><td><code>spark.memory.storageFraction</code></td><td>0.5</td><td>存储内存的比例。值越高，执行可用的内存越少，数据写到磁盘的频率越高。建议使用默认值。</td></tr><tr><td><code>spark.memory.offHeap.enabled</code></td><td>false</td><td>是否使用非堆内存。</td></tr><tr><td><code>spark.memory.offHeap.size</code></td><td>0</td><td>用于分配非堆内存的内存绝对值。该值对对内存的使用没有影响，如果想将执行的内存消耗限制在一定范围，应减小 JVM 的对内存大小。<code>spark.memory.offHeap.enabled=true</code>时，该值必须设置。</td></tr><tr><td><code>spark.memory.useLegacyMode</code></td><td>false</td><td>是否使用 spark 1.5 及以前的 legacy memory management 模式。​legacy 模式将对内存分为固定大小的分区，如果没有调优，可能会导致大量的泄露。</td></tr><tr><td><code>spark.shuffle.memoryFraction</code></td><td>0.2</td><td>废弃的。This is read only if <code>spark.memory.useLegacyMode</code> is enabled. Fraction of Java heap to use for aggregation and cogroups during shuffles. At any given time, the collective size of all in-memory maps used for shuffles is bounded by this limit, beyond which the contents will begin to spill to disk. If spills are often, consider increasing this value at the expense of <code>spark.storage.memoryFraction</code>.</td></tr><tr><td><code>spark.storage.memoryFraction</code></td><td>0.6</td><td>废弃的。This is read only if <code>spark.memory.useLegacyMode</code> is enabled. Fraction of Java heap to use for Spark's memory cache. This should not be larger than the "old" generation of objects in the JVM,which by default is given 0.6 of the heap, but you can increase it if you configure your own old generation size.</td></tr><tr><td><code>spark.storage.unrollFraction</code></td><td>0.2</td><td>(deprecated) This is read only if <code>spark.memory.useLegacyMode</code> is enabled. Fraction of <code>spark.storage.memoryFraction</code>to use for unrolling blocks in memory. This is dynamically allocated by dropping existing blocks when there is not enough free storage space to unroll the new block in its entirety.</td></tr></table>
### Execution Behavior
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.broadcast.blockSize</code></td><td>4m</td><td><code>TorrentBroadcastFactory</code>每个块的大小。太大会降低并行度，使程序执行变慢。太小会影响<code>BlockManager</code>的性能。</td></tr><tr><td><code>spark.executor.cores</code></td><td>YARN 模式下是 1 ,standalone 和 Mesos 模式下为所有可用核心数量。</td><td>每个 executor 可以使用的核心数量。如果 worker 节点有充足的核心， 在 standalone 和 Mesos 模式下允许应用在同一个 worker 节点运行多个 executor 。否则一个 worker 节点只能运行一个 executor 。</td></tr><tr><td><code>spark.default.parallelism</code></td><td>父 RDD 最大分区数量，比如 <code>reduceByKey</code> and <code>join</code> 这样的分布式 shuffle 操作。没有父 RDD 的并且操作，依赖于集群管理器：<ul><li>Local mode: 本地集器的核心数</li><li>Mesos fine grained mode: 8</li><li>Others: executor 所有核心数和 2 的较大值</li></ul></td><td>转换操作返回的 RDD 分区数量，比如 <code>join</code>，<code>reduceByKey</code> 和 <code>parallelize</code>。</td></tr><tr><td><code>spark.executor.heartbeatInterval</code></td><td>10s</td><td>executor 和驱动程序之间心跳时间间隔。心跳可以让驱动程序知道哪些 executor 活着，更新正在执行的任务信息。<code>spark.executor.heartbeatInterval</code>应小于<code>spark.network.timeout</code>。</td></tr><tr><td><code>spark.files.fetchTimeout</code></td><td>60s</td><td>使用<code>SparkContext.addFile()</code>从驱动程序获取文件的超时时间。</td></tr><tr><td><code>spark.files.useFetchCache</code></td><td>true</td><td>如果设置为 true ，获取文件将使用本地缓存。本地缓存和同一个程序的 executor 是共享的，这样可以在相同主机同时运行多个 executor 时，任务启动更快。如果设置为 false，executor 会获取各自的文件副本。为了使用 NFS 文件系统的 Spark 本地目录，可以禁用该项。</td></tr><tr><td><code>spark.files.overwrite</code></td><td>false</td><td>通过<code>SparkContext.addFile()</code>获取文件，如果文件存在并且内容不一致时是否覆盖。</td></tr><tr><td><code>spark.files.maxPartitionBytes</code></td><td>134217728 (128 MB)</td><td>读文件时单个分区打包的最大字节数。</td></tr><tr><td><code>spark.files.openCostInBytes</code></td><td>4194304 (4 MB)</td><td>预估打开文件的成本，同时读取文件的字节数量。一个分区放置多个文件时使用。超过 4MB ，小文件的分区会比大文件的快。</td></tr><tr><td><code>spark.hadoop.cloneConf</code></td><td>false</td><td>如果设置为 true ，会为每一个任务克隆新的 Hadoop <code>Configuration</code> 对象。</td></tr><tr><td><code>spark.hadoop.validateOutputSpecs</code></td><td>true</td><td>如果设置为 true ，校验在使用 saveAsHadoopFile 或其他动作时的输出规范，如校验目录是否存在。除非要兼容 Spark 以前的版本，否则不推荐禁用该项。对于使用 spark streming 的 StreamingContext 创建作业，在检查恢复时，数据需要被重写到已存在的目录，该项会被忽略。</td></tr><tr><td><code>spark.storage.memoryMapThreshold</code></td><td>2m</td><td>从磁盘读取块时映射到 Spark 内存的块的大小。这可以防止映射到很小的块。通常，在操作系统关闭块和降低页大小时内存映射开销很大。</td></tr></table>
### Networking
<table class="table table-bordered table-striped table-condensed"><tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr><tr><td><code>spark.rpc.message.maxSize</code></td><td>128</td><td>executor 和驱动程序之间发送消息的最大大小。如果运行的作业包含几千个 map 和 reduce 任务，可以调大该值。</td></tr><tr><td><code>spark.blockManager.port</code></td><td>(random)</td><td>executor 和驱动程序上所有块管理器被监听的接口。</td></tr><tr><td><code>spark.driver.blockManager.port</code></td><td>(value of spark.blockManager.port)</td><td>块管理器的指定驱动程序监听端口，不能使用和 executor 相同的配置。</td></tr><tr><td><code>spark.driver.bindAddress</code></td><td>(value of spark.driver.host)</td><td>主机名或 IP 地址用于绑定 socket 。该配置会覆盖环境变量<code>SPARK_LOCAL_IP</code>。允许设置不同的地址从本地到 executor 或 外部系统。为了能够正常工作，驱动程序使用的不同的接口需要从容器主机转发。</td></tr><tr><td><code>spark.driver.host</code></td><td>(local hostname)</td><td>驱动程序的主机名或 IP 地址。用于 executor 和 standalone 主节点之间的通信。</td></tr><tr><td><code>spark.driver.port</code></td><td>(random)</td><td>驱动程序的端口。用于 executor 和 standalone 主节点之间的通信。</td></tr><tr><td><code>spark.network.timeout</code></td><td>120s</td><td>网络默认超时时间。如果<code>spark.core.connection.ack.wait.timeout</code>,<code>spark.storage.blockManagerSlaveTimeoutMs</code>, <code>spark.shuffle.io.connectionTimeout</code>,<code>spark.rpc.askTimeout</code> or <code>spark.rpc.lookupTimeout</code>没有配置，可以用该项代替。</td></tr><tr><td><code>spark.port.maxRetries</code></td><td>16</td><td>放弃端口前最大重试次数。给一个端口指定一个非 0 值，每次重试会将原来的值 +1 。从开始端口到从端口号加最大重试次数范围的端口号都会尝试。</td></tr><tr><td><code>spark.rpc.numRetries</code></td><td>3</td><td>RPC 任务放弃前重试次数。</td></tr><tr><td><code>spark.rpc.retry.wait</code></td><td>3s</td><td>RPC 询问重试的间隔时间。</td></tr><tr><td><code>spark.rpc.askTimeout</code></td><td><code>spark.network.timeout</code></td><td>RPC 询问超时时间。</td></tr><tr><td><code>spark.rpc.lookupTimeout</code></td><td>120s</td><td>RPC 寻找远程断点的超时时间。</td></tr></table>
### Scheduling
<table class="table table-bordered table-striped table-condensed">
<tr><th>Property Name</th><th>Default</th><th>Meaning</th>
</tr><tr><td><code>spark.cores.max</code></td><td>(not set)</td>
<td>在 standalone 模式和 Mesos 粗粒度模式下，程序请求的 CPU 最大核心总数（不是每个机器的核心数量）。不设置该值，standalone 模式默认为<code>spark.deploy.defaultCores</code>的值，Mesos 默认是所有可用核心数。
</td></tr><tr><td><code>spark.locality.wait</code></td><td>3s</td>
<td>放弃启动数据在本地的任务并启动一个非本地节点任务的等待时间。这和跳过多个本地级别（process-local, node-local, rack-local 等）的是相同的。
通过设置<code>spark.locality.wait.node</code>可以自定义每个级别的等待时间。默认时间通常可以正常工作，如果你的任务很长且不在本地执行，可以增大该值。
</td></tr><tr><td><code>spark.locality.wait.node</code></td><td>spark.locality.wait</td>
<td>自定义本地节点的位置。例如，设置 0 以跳过本地节点并立刻搜索机架位置。</td></tr><tr><td><code>spark.locality.wait.process</code></td><td>spark.locality.wait</td>
<td>自定义本地进程的位置。会影响特殊的 executor 进程中试图访问缓存数据的任务。
</td></tr><tr><td><code>spark.locality.wait.rack</code></td><td>spark.locality.wait</td>
<td>自定义本地机架的位置。</td></tr><tr><td><code>spark.scheduler.maxRegisteredResourcesWaitingTime</code></td>
<td>30s</td><td>调度开始前等待资源注册的最大时间。</td></tr><tr>
<td><code>spark.scheduler.minRegisteredResourcesRatio</code></td>
<td>YARN 模式为 0.8，standalone 模式和 Mesos 粗粒度模式为 0.0</td>
<td>调度开始前等待资源注册最小比例（注册的资源/期望注册的所有资源）。指定 0.0 到 1.0 之间的浮点数。不管是否到达设定值，最大等待时间是通过<code>spark.scheduler.maxRegisteredResourcesWaitingTime</code>控制的。
</td></tr><tr><td><code>spark.scheduler.mode</code></td>
<td>FIFO</td><td>任务提交到相同的 SparkContext 的模式。多用户使用时可以设置<code>FAIR</code>来公平共享模式来代替 FIFO 。
</td></tr><tr><td><code>spark.scheduler.revive.interval</code></td>
<td>1s</td><td>调度器恢复 worker 节点资源用来运行任务的时间。
</td></tr><tr><td><code>spark.blacklist.enabled</code></td><td>false</td>
<td>如果设置为 true ，防止 Spark 从任务失败次数过多被加入黑名单的 executor 上调度任务。黑名单算法通过<code>spark.blacklist</code>配置项决定。
</td></tr><tr><td><code>spark.blacklist.task.maxTaskAttemptsPerExecutor</code></td><td>1</td>
<td>（实验性）对于一个任务，executor 被加入黑名单前 executor 上的重试次数。
</td></tr><tr><td><code>spark.blacklist.task.maxTaskAttemptsPerNode</code></td><td>2</td>
<td>（实验性）对于一个任务，节点被加入黑名单前节点上的重试次数。
</td></tr><tr><td><code>spark.blacklist.stage.maxFailedTasksPerExecutor</code></td>
<td>2</td>
<td>（实验性）被加入黑名单前有多少个不同的任务在同一个阶段同一个 executor 上失败。
</td></tr><tr><td><code>spark.blacklist.stage.maxFailedExecutorsPerNode</code></td>
<td>2</td>
<td>（实验性）在整个节点被标记为失败前有多少个不同的 executor 被加入黑名单。
</td></tr>
<tr><td><code>spark.speculation</code></td>
<td>false</td>
<td>如果设置为 true ，执行任务执行情况推测。意味着如果一个阶段的一个或多个任务运行过慢，被重新启动。
</td>
</tr>
<tr>
<td><code>spark.speculation.interval</code></td>
<td>100ms</td>
<td>多久检查一次任务执行预估。
</td>
</tr>
<tr>
<td><code>spark.speculation.multiplier</code></td>
<td>1.5</td>
<td>任务执行速度比预估的中位数慢多少倍。
</td>
</tr>
<tr>
<td><code>spark.speculation.quantile</code></td>
<td>0.75</td>
<td>执行任务预估前必须完成的任务比例。
</td>
</tr>
<tr>
<td><code>spark.task.cpus</code></td>
<td>1</td>
<td>每个任务分配的核心数。
</td>
</tr>
<tr>
<td><code>spark.task.maxFailures</code></td>
<td>4</td>
<td>一个具体的任务允许失败的次数。不同任务的失败总数不会导致作业失败。应该设置大于等于 1 ，允许重试次数比该值少 1 。</td>
</tr>
<tr>
<td><code>spark.task.reaper.enabled</code></td>
<td>false</td>
<td>
Enables monitoring of killed / interrupted tasks. When set to true, any task which is killed will be
monitored by the executor until that task actually finishes executing. See the other <code>spark.task.reaper.*</code>
configurations for details on how to control the exact behavior of this monitoring. When set to false (the
default), task killing will use an older code path which lacks such monitoring.
</td>
</tr>
<tr>
<td><code>spark.task.reaper.pollingInterval</code></td>
<td>10s</td>
<td>
When <code>spark.task.reaper.enabled = true</code>, this setting controls the frequency at which executors
will poll the status of killed tasks. If a killed task is still running when polled then a warning will be
logged and, by default, a thread-dump of the task will be logged (this thread dump can be disabled via the
<code>spark.task.reaper.threadDump</code> setting, which is documented below).
</td>
</tr>
<tr>
<td><code>spark.task.reaper.threadDump</code></td>
<td>true</td>
<td>
When <code>spark.task.reaper.enabled = true</code>, this setting controls whether task thread dumps are
logged during periodic polling of killed tasks. Set this to false to disable collection of thread dumps.
</td>
</tr>
<tr>
<td><code>spark.task.reaper.killTimeout</code></td>
<td>-1</td>
<td>
When <code>spark.task.reaper.enabled = true</code>, this setting specifies a timeout after which the
executor JVM will kill itself if a killed task has not stopped running. The default value, -1, disables this
mechanism and prevents the executor from self-destructing. The purpose of this setting is to act as a
safety-net to prevent runaway uncancellable tasks from rendering an executor unusable.
</td>
</tr>
</table>

### Dynamic Allocation
<table class="table table-bordered table-striped table-condensed">
<tr>
<th>Property Name</th>
<th>Default</th>
<th>Meaning</th>
</tr>
<tr>
<td><code>spark.dynamicAllocation.enabled</code></td>
<td>false</td>
<td>
Whether to use dynamic resource allocation, which scales the number of executors registered with this
application up and down based on the workload. For more detail, see the description <a
    href="job-scheduling.html#dynamic-resource-allocation">here</a>. <br/><br/> This requires <code>spark.shuffle.service.enabled</code>
to be set. The following configurations are also relevant: <code>spark.dynamicAllocation.minExecutors</code>,
<code>spark.dynamicAllocation.maxExecutors</code>, and <code>spark.dynamicAllocation.initialExecutors</code>
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.executorIdleTimeout</code></td>
<td>60s</td>
<td>
If dynamic allocation is enabled and an executor has been idle for more than this duration, the executor
will be removed. For more detail, see this <a href="job-scheduling.html#resource-allocation-policy">description</a>.
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.cachedExecutorIdleTimeout</code></td>
<td>infinity</td>
<td>
If dynamic allocation is enabled and an executor which has cached data blocks has been idle for more than
this duration, the executor will be removed. For more details, see this <a
    href="job-scheduling.html#resource-allocation-policy">description</a>.
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.initialExecutors</code></td>
<td><code>spark.dynamicAllocation.minExecutors</code></td>
<td>
Initial number of executors to run if dynamic allocation is enabled. <br/><br/> If `--num-executors` (or
`spark.executor.instances`) is set and larger than this value, it will be used as the initial number of
executors.
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.maxExecutors</code></td>
<td>infinity</td>
<td>
Upper bound for the number of executors if dynamic allocation is enabled.
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.minExecutors</code></td>
<td>0</td>
<td>
Lower bound for the number of executors if dynamic allocation is enabled.
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.schedulerBacklogTimeout</code></td>
<td>1s</td>
<td>
If dynamic allocation is enabled and there have been pending tasks backlogged for more than this duration,
new executors will be requested. For more detail, see this <a
    href="job-scheduling.html#resource-allocation-policy">description</a>.
</td>
</tr>
<tr>
<td><code>spark.dynamicAllocation.sustainedSchedulerBacklogTimeout</code></td>
<td><code>schedulerBacklogTimeout</code></td>
<td>
Same as <code>spark.dynamicAllocation.schedulerBacklogTimeout</code>, but used only for subsequent executor
requests. For more detail, see this <a href="job-scheduling.html#resource-allocation-policy">description</a>.
</td>
</tr>
</table>

### Security
<table class="table table-bordered table-striped table-condensed">
<tr>
<th>Property Name</th>
<th>Default</th>
<th>Meaning</th>
</tr>
<tr>
<td><code>spark.acls.enable</code></td>
<td>false</td>
<td>
Whether Spark acls should be enabled. If enabled, this checks to see if the user has access permissions to
view or modify the job. Note this requires the user to be known, so if the user comes across as null no
checks are done. Filters can be used with the UI to authenticate and set the user.
</td>
</tr>
<tr>
<td><code>spark.admin.acls</code></td>
<td>Empty</td>
<td>
Comma separated list of users/administrators that have view and modify access to all Spark jobs. This can be
used if you run on a shared cluster and have a set of administrators or devs who help debug when things do
not work. Putting a "*" in the list means any user can have the privilege of admin.
</td>
</tr>
<tr>
<td><code>spark.admin.acls.groups</code></td>
<td>Empty</td>
<td>
Comma separated list of groups that have view and modify access to all Spark jobs. This can be used if you
have a set of administrators or developers who help maintain and debug the underlying infrastructure.
Putting a "*" in the list means any user in any group can have the privilege of admin. The user groups are
obtained from the instance of the groups mapping provider specified by
<code>spark.user.groups.mapping</code>. Check the entry <code>spark.user.groups.mapping</code> for more
details.
</td>
</tr>
<tr>
<td><code>spark.user.groups.mapping</code></td>
<td><code>org.apache.spark.security.ShellBasedGroupsMappingProvider</code></td>
<td>
The list of groups for a user are determined by a group mapping service defined by the trait
org.apache.spark.security.GroupMappingServiceProvider which can configured by this property. A default unix
shell based implementation is provided
<code>org.apache.spark.security.ShellBasedGroupsMappingProvider</code> which can be specified to resolve a
list of groups for a user. <em>Note:</em> This implementation supports only a Unix/Linux based environment.
Windows environment is currently <b>not</b> supported. However, a new platform/protocol can be supported by
implementing the trait <code>org.apache.spark.security.GroupMappingServiceProvider</code>.
</td>
</tr>
<tr>
<td><code>spark.authenticate</code></td>
<td>false</td>
<td>
Whether Spark authenticates its internal connections. See <code>spark.authenticate.secret</code> if not
running on YARN.
</td>
</tr>
<tr>
<td><code>spark.authenticate.secret</code></td>
<td>None</td>
<td>
Set the secret key used for Spark to authenticate between components. This needs to be set if not running on
YARN and authentication is enabled.
</td>
</tr>
<tr>
<td><code>spark.authenticate.enableSaslEncryption</code></td>
<td>false</td>
<td>
Enable encrypted communication when authentication is enabled. This is supported by the block transfer
service and the
RPC endpoints.
</td>
</tr>
<tr>
<td><code>spark.network.sasl.serverAlwaysEncrypt</code></td>
<td>false</td>
<td>
Disable unencrypted connections for services that support SASL authentication. This is currently supported
by the external shuffle service.
</td>
</tr>
<tr>
<td><code>spark.core.connection.ack.wait.timeout</code></td>
<td><code>spark.network.timeout</code></td>
<td>
How long for the connection to wait for ack to occur before timing out and giving up. To avoid unwilling
timeout caused by long pause like GC, you can set larger value.
</td>
</tr>
<tr>
<td><code>spark.core.connection.auth.wait.timeout</code></td>
<td>30s</td>
<td>
How long for the connection to wait for authentication to occur before timing out and giving up.
</td>
</tr>
<tr>
<td><code>spark.modify.acls</code></td>
<td>Empty</td>
<td>
Comma separated list of users that have modify access to the Spark job. By default only the user that
started the Spark job has access to modify it (kill it for example). Putting a "*" in the list means any
user can have access to modify it.
</td>
</tr>
<tr>
<td><code>spark.modify.acls.groups</code></td>
<td>Empty</td>
<td>
Comma separated list of groups that have modify access to the Spark job. This can be used if you have a set
of administrators or developers from the same team to have access to control the job. Putting a "*" in the
list means any user in any group has the access to modify the Spark job. The user groups are obtained from
the instance of the groups mapping provider specified by <code>spark.user.groups.mapping</code>. Check the
entry <code>spark.user.groups.mapping</code> for more details.
</td>
</tr>
<tr>
<td><code>spark.ui.filters</code></td>
<td>None</td>
<td>
Comma separated list of filter class names to apply to the Spark web UI. The filter should be a standard <a
    href="http://docs.oracle.com/javaee/6/api/javax/servlet/Filter.html"> javax servlet Filter</a>.
Parameters to each filter can also be specified by setting a java system property of: <br/> <code>spark.&lt;class
name of filter&gt;.params='param1=value1,param2=value2'</code><br/> For example: <br/> <code>-Dspark.ui.filters=com.test.filter1</code>
<br/> <code>-Dspark.com.test.filter1.params='param1=foo,param2=testing'</code></td>
</tr>
<tr>
<td><code>spark.ui.view.acls</code></td>
<td>Empty</td>
<td>
Comma separated list of users that have view access to the Spark web ui. By default only the user that
started the Spark job has view access. Putting a "*" in the list means any user can have view access to this
Spark job.
</td>
</tr>
<tr>
<td><code>spark.ui.view.acls.groups</code></td>
<td>Empty</td>
<td>
Comma separated list of groups that have view access to the Spark web ui to view the Spark Job details. This
can be used if you have a set of administrators or developers or users who can monitor the Spark job
submitted. Putting a "*" in the list means any user in any group can view the Spark job details on the Spark
web ui. The user groups are obtained from the instance of the groups mapping provider specified by <code>spark.user.groups.mapping</code>.
Check the entry <code>spark.user.groups.mapping</code> for more details.
</td>
</tr>
</table>

### Encryption
<table class="table table-bordered table-striped table-condensed">
<tr>
<th>Property Name</th>
<th>Default</th>
<th>Meaning</th>
</tr>
<tr>
<td><code>spark.ssl.enabled</code></td>
<td>false</td>
<td>
Whether to enable SSL connections on all supported protocols. <br/>When <code>spark.ssl.enabled</code> is
configured, <code>spark.ssl.protocol</code> is required. <br/>All the SSL settings like
<code>spark.ssl.xxx</code> where <code>xxx</code> is a particular configuration property, denote the global
configuration for all the supported protocols. In order to override the global configuration for the
particular protocol, the properties must be overwritten in the protocol-specific namespace. <br/>Use <code>spark.ssl.YYY.XXX</code>
settings to overwrite the global configuration for particular protocol denoted by <code>YYY</code>. Example
values for <code>YYY</code> include <code>fs</code>, <code>ui</code>, <code>standalone</code>, and <code>historyServer</code>.
See <a href="security.html#ssl-configuration">SSL Configuration</a> for details on hierarchical SSL
configuration for services.
</td>
</tr>
<tr>
<td><code>spark.ssl.enabledAlgorithms</code></td>
<td>Empty</td>
<td>
A comma separated list of ciphers. The specified ciphers must be supported by JVM. The reference list of
protocols one can find on <a
    href="https://blogs.oracle.com/java-platform-group/entry/diagnosing_tls_ssl_and_https">this</a> page.
Note: If not set, it will use the default cipher suites of JVM.
</td>
</tr>
<tr>
<td><code>spark.ssl.keyPassword</code></td>
<td>None</td>
<td>
A password to the private key in key-store.
</td>
</tr>
<tr>
<td><code>spark.ssl.keyStore</code></td>
<td>None</td>
<td>
A path to a key-store file. The path can be absolute or relative to the directory where the component is
started in.
</td>
</tr>
<tr>
<td><code>spark.ssl.keyStorePassword</code></td>
<td>None</td>
<td>
A password to the key-store.
</td>
</tr>
<tr>
<td><code>spark.ssl.keyStoreType</code></td>
<td>JKS</td>
<td>
The type of the key-store.
</td>
</tr>
<tr>
<td><code>spark.ssl.protocol</code></td>
<td>None</td>
<td>
A protocol name. The protocol must be supported by JVM. The reference list of protocols one can find on <a
    href="https://blogs.oracle.com/java-platform-group/entry/diagnosing_tls_ssl_and_https">this</a> page.
</td>
</tr>
<tr>
<td><code>spark.ssl.needClientAuth</code></td>
<td>false</td>
<td>
Set true if SSL needs client authentication.
</td>
</tr>
<tr>
<td><code>spark.ssl.trustStore</code></td>
<td>None</td>
<td>
A path to a trust-store file. The path can be absolute or relative to the directory where the component is
started in.
</td>
</tr>
<tr>
<td><code>spark.ssl.trustStorePassword</code></td>
<td>None</td>
<td>
A password to the trust-store.
</td>
</tr>
<tr>
<td><code>spark.ssl.trustStoreType</code></td>
<td>JKS</td>
<td>
The type of the trust-store.
</td>
</tr>
</table>