# 【Kettle从零开始】第八弹之Kettle变量参数传递介绍_RotKang-CSDN博客_kettle 传递参数
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[RotKang](https://blog.csdn.net/yvigmmwfn) 2014-03-11 13:24:41 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)
 63587 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)
 收藏  19 

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

对于 ETL 参数传递是一个很重要的环节，因为参数的传递会涉及到业务数据是如何抽取。下面我为大家举例一个简单的需求。

需求说明：需要抽取昨天的数据装载到目标表中。

1、  参数作用域？

答：Kettle 中参数大致可分为两类：一类是全局参数，一类是局部参数。

2、  参数如何定义？

答：

A：全局参数定义是通过当前用户下. kettle 文件夹中的 kettle.properties 文件来定义。定义方式是采用键 \\= 值对方式来定，如：start_date=20130101

**注：在配置全局变量时需要重启 Kettle 才会生效。** 

B：局部参数变量是通过 “Set Variables” 与“Get Variables”方式来设置。

**注：在 “Set Variables” 时在当前转换当中是不能马上使用，需要在作业中的下一步骤中使用，大家可以思考下什么时候用到“Set Variables”，什么时候用到“GetVariables”**

**参数使用优先级顺序：全局变量--> 局部变量--> 本身变量**

3、  参数如何使用？

答：Kettle 中参数使用方法有两种：一种是 %% 变量名 %%，一种是 ${变量名}。这两种方法变量数据类型都是数字类型。

**注：在 SQL 中使用变量时需要把 “是否替换变量” 勾选上，否则无法使变量生效。** 

了解了这些参数概念后，我们需要使用工具实现需求。

1、  创建相应的参数变量设置转换 (Set_Param.ktr) 与数据流抽取转换 (RotKang_Test01.ktr)。

问：通过什么方式来获取昨天日期作为参数？

答：我们可以通过 “获取系统信息” 组件获取昨天日期，再通过字段选择转换成 yyyy-mm-dd 格式，最后设置成变量，设置参数变量为 ${YESTERDAY}。如下图：

![](https://img-my.csdn.net/uploads/201403/11/1394515310_9911.jpg)

(图 8.0)

获取相关的日期参数，昨天日期。

![](https://img-my.csdn.net/uploads/201403/11/1394515310_5847.jpg)

(图 8.1)

由于获取日期是到时分秒，通过字段选择转换成年月日格式。

![](https://img-my.csdn.net/uploads/201403/11/1394515309_1499.jpg)

(图 8.2)

设置参数，点确定时会提示大致意思是 “设置的参数不能在当前转换中使用”。

![](https://img-my.csdn.net/uploads/201403/11/1394515317_3377.jpg)

(图 8.2)

2、  由于数据流抽取 (RotKang_Test01.ktr) 转换上几弹已经完成，但是还没有设置参数使用，设置参数后如下图：

![](https://img-my.csdn.net/uploads/201403/11/1394515309_7806.jpg)

(图 8.3)

**注：在使用参数时需要勾选 “替换 SQL 语句里的变量 “**

3、最后使用 Job 把整个流程串连起来，如下图：

![](https://img-my.csdn.net/uploads/201403/11/1394515309_8270.jpg)

(图 8.4)

最后测试 RotKang_Test.kjb 作业运行结果。 
 [https://blog.csdn.net/rotkang/article/details/21008271](https://blog.csdn.net/rotkang/article/details/21008271)
