title: Oracle 分析函数 lag over
date: 2015-08-15 9:53:41
category:
	- Oracle
tag:
	- oracle
	- 数据库
---

### 定义
先上一张官网给的图片定义 

![](/images/oracle-lag-01.gif)

<!--more-->

用此函数，你可以一次性从表中查询多条数据，而不需要将此表自连接。
它返回表中的当前行的前offset行的指定列值。

语法

--- LAG(value_expr [, offset ] [, default ])    OVER ([ query_partition_clause ] order_by_clause)

参数说明

+ **`value_expr`** 值表达式，通常是字段，也可是是表达式。但不能是分析函数
+ **`offset`** 偏移，可选参数，是表中与当前行的物理偏移度，如果省略，默认值为1，既当前行的前`offset`行。
+ **`default`** 可选参数，如果offset参数指向超出了表的范围，就返回这个值，默认值为null。
+ **`over`**  理解成在一个结果集范围内，如果后面的 `partition by`为空，那么就是当前的结果集范围内。
+ **`query_partition_clause`**  分区语句，对结果集合分区的语句，是可选的，如果没有就是所有的一个分区。
+ **`Order_by_clause`** 排序语句 必须需要 ,形如`order by xxx desc/asc`

实际用到的例子：
需求：报表需要展示 航空公司在某个时间区间内的货物重量、费用值。本例是2015-08-02～2015-08-05，做法就是0805的值减去0802的值
查询结果如下图：

~~~ sql
select carrier,
       bill_date,
       total_wt,
       lag(total_wt, 1) over(partition by carrier order by carrier) as total_wt_pre,
~~~
![](/images/oracle-lag-02.png)

`partition by carrier` 根据承运人分组，即每组承运人都重新开始取值。可以看到图中`total_wt_pre` 列有很多空值。就是按`carrier`分组的第一条

最后查询结果，只要取 `BILL_DATE=20150805` 的数据，然后 `TOTAL_WT_PRE-TOTAL_WT` 就可以了。

与之对应的是 `lead()` `over()` 函数，即取当前行的后`offset`行。

这两个函数在计算同比环比的时候很有用。


[官网文档](http://docs.oracle.com/cd/B19306_01/server.102/b14200/functions070.htm)


