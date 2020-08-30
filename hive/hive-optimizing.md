* 性能优化
* 语言特性
  * GROUPING _SETS：是_`GROUP BY`_子句的一个选项。_`GROUPING SETS`_在同一查询中定义多个分组集。_
    * 定义：`GROUPING SETS`是 `GROUP BY`子句的一个选项。GROUPING SETS在同一查询中定义多个分组集
    * 语法：
      ```sql
      SELECT
          c1,
          c2,
          GROUPING__ID()
          aggregate (c3)
      FROM
          table
      GROUP BY
          GROUPING SETS (
              (c1, c2),
              (c1),
              (c2),
              ()
      )
      ```
    * 特性：根据根据GROUPING\_\_ID确定分组，计算方式为GROUP BY字段倒序排列二进制顺序，ID值转为十进制
    * 优点：此查询比UNION ALL方式查询更具可读性和执行速度，因为数据库系统不必多次读取库存表。
    * 注意：Spark SQL中，GROUPING\_\_ID\(\)值与Hive中相反，计算方式为GROUP BY字段正序排列二进制顺序   
      ```sql
      添加参数：spark.sql.support.hive.groupingId.enable=true 可保持与Hive一致
      ```
    * 类比：Hive中提供类似函数还有`ROLLUP`（上卷）与`CUBE`
      ```
      group by a,b WITH rollup:      group by a, b grouping sets ( (a,b),a,b,() )
      group by a, b, c WITH CUBE:    group by a, b, c grouping sets ( (a, b, c), (a, b), (b, c), (a, c),(a), (b), (c), ( ))
      ```



