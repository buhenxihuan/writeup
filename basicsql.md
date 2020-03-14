# Basicsql

这是一道简单的sql注入题，考察的就是对于sql注入的基本功，整个页面只有一个搜索框，注入点一目了然

输入了一下1' or 1=1# 发现查找成功，即该网站未对#进行过滤，那么接下来则变得简单

开始使用order by 语句猜字段数，在4时查询失败，可知有3个可显示位，接下来看是否过滤union select字段，结果发现没有过滤

于是使用1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() #语句获取表名![](.\image\basicsql-1.png)



