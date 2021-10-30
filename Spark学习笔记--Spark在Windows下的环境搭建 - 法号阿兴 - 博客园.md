# Spark学习笔记--Spark在Windows下的环境搭建 - 法号阿兴 - 博客园
　　本文主要是讲解 Spark 在 Windows 环境是如何搭建的

#### 1、1 下载 JDK

　　首先需要安装 JDK，并且将环境变量配置好，如果已经安装了的老司机可以忽略。**JDK**（全称是 JavaTM Platform Standard Edition Development Kit）的安装，去 Oracle 官网下载，下载地址是[Java SE Downloads](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 。

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927203501294-1676564950.png)

　　上图中两个用红色标记的地方都是可以点击的，点击进去之后可以看到这个最新版本的一些更为详细的信息，如下图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927203527903-211275111.png)

　　下载完之后，我们安装就可以直接 JDK，JDK 在 windows 下的安装非常简单，按照正常的软件安装思路去双击下载得到的 exe 文件，然后设定你自己的安装目录（这个安装目录在设置环境变量的时候需要用到）即可。

#### 1、2 JDK 环境变量设置

　　接下来设置相应的环境变量，设置方法为：在桌面右击【计算机】－－【属性】－－【高级系统设置】，然后在系统属性里选择【高级】－－【环境变量】，然后在系统变量中找到**“Path”**变量，并选择**“编辑”**按钮后出来一个对话框，可以在里面添加上一步中所安装的 JDK 目录下的 bin 文件夹路径名，我这里的 bin 文件夹路径名是：C:\\Program Files\\Java\\jre1.8.0_92\\bin，所以将这个添加到 path 路径名下，注意用英文的分号 “;” 进行分割。如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927203548137-568948097.png)

　　这样设置好后，便可以在任意目录下打开的 cmd 命令行窗口下运行下面命令。查看是否设置成功。

　　观察是否能够输出相关 java 的版本信息，如果能够输出，说明 JDK 安装这一步便全部结束了。如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927203600403-1256066982.png)

　　我们从官网：[http://www.scala-lang.org/](http://www.scala-lang.org/) 下载 Scala，最新的版本为 2.12.3，如图所示

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927203647137-410377883.png)

因为我们是在 Windows 环境下，这也是本文的目的，我们选择对应的 Windows 版本下载，如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204415700-300655739.png)

　　下载得到 Scala 的 msi 文件后，可以双击执行安装。安装成功后，默认会将 Scala 的 bin 目录添加到**PATH**系统变量中去（如果没有，和上面 JDK 安装步骤中类似，将 Scala 安装目录下的 bin 目录路径，添加到系统变量**PATH**中），为了验证是否安装成功，开启一个新的 cmd 窗口，输入`scala`然后回车，如果能够正常进入到 Scala 的交互命令环境则表明安装成功。如下图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204440590-984733745.png)

备注：如果不能显示版本信息，并且未能进入 Scala 的交互命令行，通常有两种可能性：   
1、Path 系统变量中未能正确添加 Scala 安装目录下的 bin 文件夹路径名，按照 JDK 安装中介绍的方法添加即可。   
2、Scala 未能够正确安装，重复上面的步骤即可。

我们到 Spark 官网进行下载：[http://spark.apache.org/](http://spark.apache.org/) ，我们选择带有 Hadoop 版本的 Spark，如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204611419-739525306.png)

　　下载后得到了大约 200M 的文件： spark-2.2.0-bin-hadoop2.7

　　这里使用的是 Pre-built 的版本，意思就是已经编译了好了，下载来直接用就好，Spark 也有源码可以下载，但是得自己去手动编译之后才能使用。下载完成后将文件进行解压（可能需要解压两次），最好解压到一个盘的根目录下，并重命名为 Spark，简单不易出错。并且需要注意的是，在 Spark 的文件目录路径名中，不要出现空格，类似于 “Program Files” 这样的文件夹名是不被允许的。我们在 C 盘新建一个 Spark 文件夹存放，如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204821340-910193104.png)

　　解压后基本上就差不多可以到 cmd 命令行下运行了。但这个时候每次运行 spark-shell（spark 的命令行交互窗口）的时候，都需要先`cd`到 Spark 的安装目录下，比较麻烦，因此可以将 Spark 的 bin 目录添加到系统变量**PATH**中。例如我这里的 Spark 的 bin 目录路径为`D:\Spark\bin`，那么就把这个路径名添加到系统变量的**PATH**中即可，方法和 JDK 安装过程中的环境变量设置一致，设置完系统变量后，在任意目录下的 cmd 命令行中，直接执行`spark-shell`命令，即可开启 Spark 的交互式命令行模式。

　　系统变量设置后，就可以在任意当前目录下的 cmd 中运行 spark-shell，但这个时候很有可能会碰到各种错误，这里主要是因为 Spark 是基于 hadoop 的，所以这里也有必要配置一个 Hadoop 的运行环境。错误如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204849512-1059219829.png)

接下来，我们还需要安装 Hadoop。

　　在[Hadoop Releases](https://archive.apache.org/dist/hadoop/common/)里可以看到 Hadoop 的各个历史版本，这里由于下载的 Spark 是基于 Hadoop 2.7 的（在 Spark 安装的第一个步骤中，我们选择的是`Pre-built for Hadoop 2.7`），我这里选择 2.7.1 版本，选择好相应版本并点击后，进入详细的下载页面，如下图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204919169-1159261451.png)

　　选择图中红色标记进行下载，这里上面的 src 版本就是源码，需要对 Hadoop 进行更改或者想自己进行编译的可以下载对应 src 文件，我这里下载的就是已经编译好的版本，即图中的 “hadoop-2.7.1.tar.gz” 文件。

下载并解压到指定目录，，我这里是 C:\\Hadoop，如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927204952919-932233848.png)

然后到环境变量部分设置**HADOOP_HOME**为 Hadoop 的解压目录，如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927205023840-367821451.png)

然后再设置该目录下的 bin 目录到系统变量的**PATH**下，我这里也就是 C:\\Hadoop\\bin，如果已经添加了**HADOOP_HOME**系统变量，也可用 %HADOOP_HOME%\\bin 来指定 bin 文件夹路径名。这两个系统变量设置好后，开启一个新的 cmd 窗口，然后直接输入`spark-shell`命令。如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927205044372-2028981225.png)

　　正常情况下是可以运行成功并进入到 Spark 的命令行环境下的，但是对于有些用户可能会遇到空指针的错误。这个时候，主要是因为 Hadoop 的 bin 目录下没有 winutils.exe 文件的原因造成的。这里的解决办法是： 

　　可以去 [https://github.com/steveloughran/winutils](https://github.com/steveloughran/winutils) 选择你安装的 Hadoop 版本号，然后进入到 bin 目录下，找到`winutils.exe`文件，下载方法是点击`winutils.exe`文件，进入之后在页面的右上方部分有一个`Download`按钮，点击下载即可。 如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927205908653-574666672.png)

下载 winutils.exe 文件

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927205927278-1942154074.png)

　　将下载好`winutils.exe`后，将这个文件放入到 Hadoop 的 bin 目录下，我这里是 C:\\Hadoop\\hadoop-2.7.1\\bin。

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927210001090-1229212725.png)

在打开的 cmd 中输入 

C:\\Hadoop\\hadoop-2.7.1\\bin\\winutils.exe chmod 777 /tmp/Hive  // 修改权限，777 是获取所有权限

但是我们发现报了一些其他的错 (Linux 环境下也是会出现这个错误)

1 <console>:14: error: not found: value spark
2        import spark.implicits.\_
3               ^
4 <console>:14: error: not found: value spark
5        import spark.sql

其原因是没有权限在 spark 中写入 metastore_db 这个文件。

处理方法：我们授予 777 的权限

**Linux 环境**，我们在 root 下操作：

1 sudo chmod 777 /home/hadoop/spark 2 
3 #为了方便，可以给所有的权限 4 sudo chmod a+w /home/hadoop/spark

**window 环境下：** 

存放 Spark 的文件夹不能设为只读和隐藏，如图所示：

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927210016403-1046114626.png)

授予完全控制的权限，如图所示：

![](https://img2020.cnblogs.com/blog/506829/202101/506829-20210128185543159-304918152.png)

经过这几个步骤之后，然后再次开启一个新的 cmd 窗口，如果正常的话，应该就可以通过直接输入`spark-shell`来运行 Spark 了。正常的运行界面应该如下图所示：

![](https://img2020.cnblogs.com/blog/506829/202101/506829-20210128185630050-1103888846.png)

六、**Python 下 Spark 开发环境搭建**

 下面简单讲解 Python 下怎么搭建 Spark 环境

1、将 spark 目录下的 pyspark 文件夹（C:\\Spark\\python\\pyspark）复制到 python 安装目录 C:\\Python\\Python35\\Lib\\site-packages 里。如图所示

spark 的 pysaprk

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927210104887-2137914021.png)

将 pyspark 拷贝至 Python 的安装的 packages 目录下。

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927210116731-567571627.png)

2、然后使用 cd 命令，进入目录 D:\\python27\\Scripts，运行 pip install py4j 安装 py4j 库。如图所示：

![](https://img2020.cnblogs.com/blog/506829/202101/506829-20210128185656957-765548462.png)

如果需要在 python 中或者在类似于 IDEA IntelliJ 或者 PyCharm(笔者用的就是 PyCharm) 等 IDE 中使用 PySpark 的话，需要在系统变量中新建一个 PYTHONPATH 的系统变量，然后设置好下面变量值就可以了

PATHONPATH=%SPARK_HOME%\\python;%SPARK_HOME%\\python\\lib\\py4j-0.10.4-src.zip

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170927210144622-1962335708.png)

后面的事情就交给 PyCharm 了。

至此，Spark 在 Windows 环境下的搭建讲解已结束。

**PS：如有问题，请留言，未经允许不得私自转载，转载请注明出处：[http://www.cnblogs.com/xuliangxing/p/7279662.html](http://www.cnblogs.com/xuliangxing/p/7279662.html)**

![](https://images2017.cnblogs.com/blog/506829/201709/506829-20170922151128712-702940070.gif)

![](https://img2020.cnblogs.com/blog/506829/202006/506829-20200620010517082-1964113829.png) 
 [https://www.cnblogs.com/xuliangxing/p/7279662.html](https://www.cnblogs.com/xuliangxing/p/7279662.html)
