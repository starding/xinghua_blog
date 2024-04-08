---
title: django 中使用原生 sql
og_title: django 中使用原生 sql
date: 2016-01-07 18:47:03
tags:
---

## ORM 并非万能

从功能集上讲，Django 的 ORM 只是 SQL 的一个子集。这意味着很多使用 SQL 能实现的功能，Django ORM 无法完成，更不用说 SQL 甚至是图灵完备的了。例如，直到 1.8 版本，Django 才逐渐实现了`CASE`、`WHEN`、`IF`这些控制流。而这些内容在一些特殊类型的表操作中非常常见，比如说报表管理。

好在，Django 提供了使用原生 SQL 的接口，这样就能通过原生 SQL 来实现一些复杂的功能。

## SQL 控制流之 CASE WHEN 一个例子

现有一张档案信息表`archives`：

字段说明：
- `number` 档案号
- `type` 档案类型
- `status` 档案状态
- `company` 公司
- `branch_company` 分公司

需求是计算出表中同一`type`，同一分公司下的档案总数，和`status=01`的档案数，以及它占档案总数的比值。

当然，使用编程语言也可以实现这个功能，但是会比较复杂。这个时候可以使用`CASE WHEN`语句来精确控制表中同一字段下，不同内容的选择。

```sql
SELECT 
      COUNT(CASE WHEN status='01' THEN status END) AS status_01,
      COUNT(*) AS total,
      CONCAT(FORMAT(COUNT(CASE WHEN status='01' THEN status END)/
      COUNT(*)*100, 2), '%') AS percentage
FROM
      archives
GROUP BY
      status, branch_company
```

这样就可以解决上面提出的问题。因为这个表是临时构造的，结果这里就不展示了。

## 在上述基础上实现链式查询
在 Django 的 ORM 中，一个非常好用的功能就是使用链式查询，你可以不断连接 filter 等方法来过滤出想要的内容。

这在一些特定的场景中特别有用。比如在上面的表中，有时候可能想要某个分公司或中支公司下的数据，有时候又想要单一类型下的数据。如果针对每一种条件组合分别写相应的 SQL 查询的话，会非常复杂，而且有时候组合会特别多。而链式查询比较完美地解决了这个问题。

为了让原生 SQL 也能有个简单的链式查询，我们需要不断连接 where 中的条件子句。为此可以写一个简单的类来实现它：


```python
class GenQuerySQL(object):
    def __init__(self, table):
        self.table = table
        self.group_by_fields = " "
        self.where_conditions = " 1=1 "
        self.fields = " "
        self.order_by_fields = ""
    def where(self, where_condition):
        if where_condition:
            self.where_conditions += " and " + where_condition
        return self
    def add_field(self, fields):
        self.fields += " " + fields
        return self
    def group_by(self, group_by_field):
        self.group_by_fields = group_by_field
        return self
    def order_by(self, order_by_field):
        self.order_by_fields = order_by_field
        return self
    def sql(self):
        SQL = 'SELECT ' + self.fields + ' FROM ' + self.table + ' WHERE ' + self.where_conditions + ' GROUP BY ' + self.group_by_fields + ' ORDER BY ' + self.order_by_fields + ";"
        return SQL
```

这个类可以简单地模拟链式查询的功能。例如：

```python
archive_statistics = GenQuerySQL('SOME_TABLE')
raw_sql = archive_statistics.add_field('fields').where('where_condition').group_by('group_by_fields').order_by('order_by_fields').sql()
```

其中.where 可以多次连接。当然也可以使用另一种方式：先把 where 语句根据条件构造完毕，最终再拼接成 sql 语句。其思想是一样的：先过滤条件，最终再查询数据库。


2016.01 于北京回龙观