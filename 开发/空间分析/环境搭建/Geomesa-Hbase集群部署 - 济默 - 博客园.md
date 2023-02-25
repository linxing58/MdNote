# Geomesa-Hbase集群部署 - 济默 - 博客园
本文记录一下 Geomesa-Hbase 集群部署，在单机部署的基础上

[https://www.cnblogs.com/help-silence/p/12817447.html](https://www.cnblogs.com/help-silence/p/12817447.html)

1\. 搭建集群

[https://www.cnblogs.com/help-silence/p/12517442.html](https://www.cnblogs.com/help-silence/p/12517442.html)

2\. 安装 Hadoop 分布式环境

[https://www.cnblogs.com/help-silence/p/12518731.html](https://www.cnblogs.com/help-silence/p/12518731.html)

3\. 安装 Zookeeper

[https://www.cnblogs.com/help-silence/p/12523466.html](https://www.cnblogs.com/help-silence/p/12523466.html)

4\. 修改 Hbase 配置为集群模式

[https://www.cnblogs.com/help-silence/p/12524484.html](https://www.cnblogs.com/help-silence/p/12524484.html)

5.Geomesa-Hbase 不需要改动

6\. 测试环境

可以参照 Geomesa-Hbase 单机版的测试方法；  
这里我用了 idea 中的代码测试，直接运行 hbase 的 qucikstart 就可以使用，由于 maven 编译 qucikstart 一直没有编译成功，我是用 idea 中 maven 插件进行编译，  
一些没有找到的依赖包，我手动下载并导进工程 (geotools 相关的 jar 包，还有 Geomesa-Hbase 的 lib 下的所有 jar 包)，可以运行。注意运行时指定参数 zookeeper 地址和 catalog 名字。

7\. 在 bin/hbase shell list 命令查看 Hbase 内的变化

![](https://common.cnblogs.com/images/copycode.gif)

hbase(main):001:0> list
TABLE  
geomesaqucikstart  
geomesaqucikstart_gdelt_2dquickstart_attr_v5  
geomesaqucikstart_gdelt_2dquickstart_id  
geomesaqucikstart_gdelt_2dquickstart_z2_v2  
geomesaqucikstart_gdelt_2dquickstart_z3_v2 5 row(s) in 0.1270 seconds \\=> \["geomesaqucikstart", "geomesaqucikstart_gdelt_2dquickstart_attr_v5","geomesaqucikstart_gdelt_2dquickstart_id","geomesaqucikstart_gdelt_2dquickstart_z2_v2","geomesaqucikstart_gdelt_2dquickstart_z3_v2"]
hbase(main):002:0> 

![](https://common.cnblogs.com/images/copycode.gif)

大功告成~！ 
 [https://www.cnblogs.com/help-silence/p/12817519.html](https://www.cnblogs.com/help-silence/p/12817519.html)
