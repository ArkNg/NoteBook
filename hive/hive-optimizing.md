* 性能优化
* 语言特性
  * GROUPING _SETS：是_`GROUP BY`_子句的一个选项。_`GROUPING SETS`_在同一查询中定义多个分组集。_
    * 定义：`GROUPING SETS`是 `GROUP BY`子句的一个选项。GROUPING SETS在同一查询中定义多个分组集
    * 语法：
      ```SQL
      SELECT
          c1,
          c2,
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
    * 特性：
    * 优点：
    * 注意：



