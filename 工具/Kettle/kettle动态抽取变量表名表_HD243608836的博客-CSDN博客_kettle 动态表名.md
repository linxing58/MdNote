# kettle动态抽取变量表名表_HD243608836的博客-CSDN博客_kettle 动态表名
ods 平台的一个很简单的数据抽取需求：

上游系统有一个月表，每个月出上个月数据并放在新建的月表里。例如：20150401 出 3 月份表和数据 TB_B_FT_BROADBAND_201503，

20150501 出 4 月份表和数据 TB_B_FT_BROADBAND_201504。而 ods 需要每月初等他们数据出来后再抽取过来。  

需求很简单，用 kettle 最常见的表输入和输出抽取即可，但是表输入的 select 语句里面的表名需要使用变量。  

使用 job kjb 如下 完成此需求，如下图，步骤如下：

1  start

2 设置表明使用的变量：时间变量（tabledate.ktr）

3 抽取（即表输入》表输出）(TB_B_FT_BROADBAND.ktr)

![](https://img-blog.csdn.net/20150429155331988?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhb2hhaTc5OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

设置时间变量的转换如下，tabledate.ktr：

![](https://img-blog.csdn.net/20150429155228466?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhb2hhaTc5OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

时间变量设置参考：[http://blog.csdn.net/xiaohai798/article/details/41867835](http://blog.csdn.net/xiaohai798/article/details/41867835)  

 TB_B_FT_BROADBAND.ktr  

抽取 TB_B_FT_BROADBAND.ktr 如下：

![](https://img-blog.csdn.net/20150429155232008?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhb2hhaTc5OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**注意点如上 ：** 

**1 from 后面的表名 ${} 里面的即是前面设置的变量**

**2 Replace variables in script? 方框 打上勾。变量即起上作用。** 

select 语句  

1.  select
2.  LATN_ID,   
3.  MONTH_ID,   
4.  PRD_INST_ID,   
5.  SERV_NBR,   
6.  PRD_ID,   
7.  CRM_PRD_ID,   
8.  PRD_INST_STAS_ID,   
9.  PRD_INST_NAME,   
10. PRD_INST_DESC,   
11. INSTALL_DATE,   
12. ....  
13. from eda.TB_B_FT_BROADBAND\_${FILTEDATE}   
    [https://blog.csdn.net/HD243608836/article/details/79266391](https://blog.csdn.net/HD243608836/article/details/79266391)
