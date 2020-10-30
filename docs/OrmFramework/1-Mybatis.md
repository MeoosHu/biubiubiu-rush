### 1、#和$符号的区别

​		#相当于对数据加上双引号，而$相当于直接显示数据，即使用#号时，相当于是预编译，使用的是？号占位符，即使用jdbc的preparedstatement，这种方式可以防止sql注入，而$符号却不能有效防止sql注入；

​		$方式一般用于传输数据库对象，例如表名、列名；

​		MyBatis排序时使用order by 动态参数时需要注意，用$而不是#

### 2、除了if还有哪些标签？

1. 定义SQL语句：select、update、insert、delete
2. 配置对象与查询结果集：resultMap
3. 动态拼接SQL：if、foreach、choose
4. 格式化输出：where、set、trim
5. 配置关联关系：collection、association
6. 定义常量：sql

### 3、一级缓存与二级缓存？