# ETL工具kettle的性能优化
性能调优在整个项目中尤为重要。对于初级开发人员往往都不知道如何对性能进行调优。其实性能调优主要分为两个方面：一方面是硬件方面的调优，一方面是软件方面的调优。本文章主要介绍[Kettle](https://so.csdn.net/so/search?q=Kettle&spm=1001.2101.3001.7020)方面的性能调优以及效率的提升。

### 一、Kettle 组件调优

##### 1. [commit](https://so.csdn.net/so/search?q=commit&spm=1001.2101.3001.7020) size

表输出的提交记录数量（默认 1000），具体根据数量大小来修改。

![](https://img-blog.csdnimg.cn/20200923135339627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

修改前速度（7447/s）：![](https://img-blog.csdnimg.cn/20200923112715528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

修改后（7992/s）:![](https://img-blog.csdnimg.cn/20200923112821276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 2. 数据库连接调参

基于上层优化方案继续调优

```
useServerPrepStmts=false
rewriteBatchedStatements=true
useCompression=true

```

![](https://img-blog.csdnimg.cn/20200923135420167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

添加参数后（20361/s）:  
![](https://img-blog.csdnimg.cn/2020092314032823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 3. Kettle 并行

右击 “START 组件” 勾选`Run Next Entries in Parallel`  
![](https://img-blog.csdnimg.cn/20200923140440479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

选择`I understand`

![](https://img-blog.csdnimg.cn/20200923140815389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

最终图：  
![](https://img-blog.csdnimg.cn/20200923141021242.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

结果图（可以看到每秒的表输出量已经起飞\~\~~）：  
70W 的数据仅用 30S 的时间就插入完成。  
![](https://img-blog.csdnimg.cn/20200923141056651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjYwOQ==,size_16,color_FFFFFF,t_70#pic_center)

### 二、Kettle 参数调优

1.  调整 JVM 大小进行性能优化，修改 Kettle 定时任务中的 Spoon 脚本。

| 参数           | 描述                                                                                                                                           |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| -Xmx1024m    | 设置 JVM 最大可用内存为 1024M。                                                                                                                        |
| -Xms512m     | 设置 JVM 促使内存为 512m。此值可以设置与 - Xmx 相同，以避免每次垃圾回收完成后 JVM 重新分配内存。                                                                                  |
| -Xmn2g（JVM）  | 设置年轻代大小为 2G。整个 JVM 内存大小 = 年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为 64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun 官方推荐配置为整个堆的 3/8。                           |
| -Xss128（JVM） | 设置每个线程的堆栈大小。JDK5.0 以后每个线程堆栈大小为 1M，以前每个线程堆栈大小为 256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在 3000~5000 左右。 |

Spoon 参数位置：`if "%PENTAHO_DI_JAVA_OPTIONS%"==""set PENTAHO_DI_JAVA_OPTIONS="-Xms512m""-Xmx512m" "-XX:MaxPermSize=128m"`

### 三、 Kettle 其他优化

1.  尽量使用 SQL 语句、尽可能的不适用组件。
2.  Kettle 是基于 Java 制作的，尽可能的用较大的内存来启动 Kettle
3.  尽可能的使用同一主机来进行 ETL 操作（本地 - 本地 / 服务器 - 服务器），本地 - 服务器传输网络波动会有一定的影响
4.  使用数据库资源库便于对作业、转换的管理与应用。
5.  尽可能的输入少量的数据集（没用的字段不抽取）
6.  插入大量数据时把索引删掉，待数据全量插入完成后再进行创建索引。 
    [https://blog.csdn.net/weixin_43932609/article/details/108749304?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108749304-blog-111804122.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108749304-blog-111804122.pc_relevant_default&utm_relevant_index=2](https://blog.csdn.net/weixin_43932609/article/details/108749304?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108749304-blog-111804122.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108749304-blog-111804122.pc_relevant_default&utm_relevant_index=2)
