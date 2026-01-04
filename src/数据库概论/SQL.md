# SQL

> 此文档给出考试和工程上常用的规则。
> 工程上要用到什么要约束什么，还是问 ai 查文档。

SQL 语法和关键字 大小写不区分

每条 SQL 语句都以 `;` 结尾

语法描述上的一些符号约定 ↓

[]：表示"可有可无"

<>: 表示名称的含义

名称在 SQL 中是必须大小写区分的

# 模式和表

## 定义模式

`create schema [ <模式名> ] authorization <用户名>`

```sql
-- 为用户 Zhang 创建一个模式 Test，并且在其中定义一个表 Tab1
create schema Test authorization Zhang
create table Tab1(
    Col1 smallint,
    Col2 int,
    Col3 char(20),
    Col4 numeric(10, 3),
    Col5 decimal(5, 2)
    );
```

## 删除模式

`drop schema <模式名> <cascade|restrict>`

> 选取的两者必选其一
> cascade(级联) —— 表示在删除模式的同时把该模式中所有的数据库对象全部删除；
> restrict(限制) —— 表示如果该模式中已经定义了数据库对象（如表、视图等），则拒绝删除语句的执行；

## 定义表

```sql
create table <表名>（
    <列名> <数据类型> [列级完整性约束],
    <列名> <数据类型> [列级完整性约束],
    ...
    [<表级完整性约束>]
    );
```

**[列级完整性约束]**

`primary key`， 列指定主键约束

`unique`，唯一约束

`not null`，非空约束，默认值 `NULL`

`default <默认值>`，默认约束

`auto_increment`，属性值自动增加。默认情况下初始值和增量都为 1

**[表级级完整性约束]**

`primary key(<列名1>, <列名2>)`，多字段联合主键

定义外键

```sql
-- 工程上写完整
[constraint <fk_子表_主表>] foreign key <子表字段> references <主表(主键)>
-- fk_子表名_主表名 在mysql中是默认命名规范
```

![](static/Kkz9bPtqco1falxMUG6cojWsniE.png)

## 描述表

`describe <表名>;`

查看表的基本信息，包括：字段名、数据类型、是否有主键、是否有默认值等

下面的表是简单的描述：

![](static/X7t0bcakqoyHrAxkrLVcVvSKnSn.png)

- `NULL`: 表示该列是否能存储 `NULL` 值
- `Key`: 表示该列是否已编制索引，`PRI` 表示该列是主键一部分
- `Default`: 表示该列是否有默认值，如果有的话值是多少
- `Extra`: 表示可以获取的与给定列有关的附加信息

`show create table <表名>;` 查看表的详细结构

## 删除表

`drop table <表名> [restrict|cascade] `

> 默认选项是 restrict，存在依赖该表的对象，此表不能被删除
> 选择 cascade，则删除该表没有限制条件。相关的依赖对象都可能被一起删除
> *每个数据库产品，`drop` 策略有差别

# 表的查询

## 单表查询

```sql
-- 别名一般为缩写 ，内部引用，select中的别名可以作为结果
-- 完整写应该是[as <别名>]，as只是个可读性标记，可以省略
select [all|distinct] <目标列表达式> [别名], ... /*相当于投影*/
/* all —— 不去重  distinct —— 去重*/
from <表名或视图名> [别名], ... 
[where <条件表达式>] /*相当于选择*/
[group by <列名> [having <条件表达式>]] /*having作用于组，where作用于表*/
/*asc —— 升序  desc —— 降序*/
[order by <列名> [asc|desc]]
[limit <行数1> [offset <行数2>]]
```

<table>
<tr>
<td>聚集函数<br/></td><td>作用<br/></td></tr>
<tr>
<td>count(*)<br/></td><td>统计元组个数<br/></td></tr>
<tr>
<td>count([distinct|all]<列名>)<br/></td><td>统计一列中值的个数<br/></td></tr>
<tr>
<td>sum([distinct|all]<列名>)<br/></td><td>计算一列值的总和（此列必须是数值列）<br/></td></tr>
<tr>
<td>avg([distinct|all]<列名>)<br/></td><td>计算一列值的平均值（此列必须是数值列）<br/></td></tr>
<tr>
<td>max([distinct|all]<列名>)<br/></td><td>求一列值中的最大值<br/></td></tr>
<tr>
<td>min([distinct|all]<列名>)<br/></td><td>求一列值中的最小值<br/></td></tr>
</table>

`where` 子句不能用聚集函数作为条件表达式，要么就用子查询

`group by` 子句 常和 `select` 中用聚集函数绑定 ；若只是目标列，分组后只会查分组的第一行

## 连接查询

理论是关系代数的连接 (等值连接，非等值连接，自然连接，（左/右）外连接  * 见教材 p50)

`from` 多个表，where 子句里写条件（示例见教材 p92）

外连接特殊，语法是 `[left|right] join <连接的表> on <条件表达式>`

```sql
-- t_emp(id, name, deptId)
-- t_dept(deptId, name)
-- 查询：员工 + 部门名
SELECT e.id, e.name, d.name AS dept_name
FROM t_emp e
JOIN t_dept d -- 准备拼部门表，完整应该是INNER JOIN，缩写 JOIN，正常外连接，两边都不保留NULL值
ON e.deptId = d.deptId; -- 只有匹配的行才能拼上
-- 左外连接
-- 就算没有部门的员工也显示，非常常用
SELECT e.id, e.name, d.name AS dept_name
FROM t_emp e
LEFT JOIN t_dept d -- 完整 LEFT OUTER JOIN，缩写 LEFT JOIN
ON e.deptId = d.deptId;
```

## 嵌套查询

`where` 子查询（最常见）

```sql
-- 查“在技术部的员工”
SELECT *
FROM t_emp
WHERE deptId = (
  SELECT deptId
  FROM t_dept
  WHERE name = '技术部'
);
```

`from` 子查询（当临时表用，最重要）

```sql
-- 给出每个部门人数 > 5 的部门
SELECT *
FROM (
  SELECT deptId, COUNT(*) AS cnt
  FROM t_emp
  GROUP BY deptId
) t
WHERE t.cnt > 5;
```

查“存不存在”，`where exists(子查询)`

```sql
-- 查“有员工的部门”
SELECT *
FROM t_dept d
WHERE EXISTS (
  SELECT 1
  FROM t_emp e
  WHERE e.deptId = d.deptId
);
```

“我需要一个**中间结果再处理**” → 子查询（FROM）

“我只关心有没有匹配” → EXISTS

“我要拼字段一起展示” → JOIN

“逻辑复杂但想分层写清楚” → 子查询

要注意的是子查询不要写出来性能开销太大，清晰且易维护

## 集合查询

对于多条 `select` 的结果集之间的集合运算（并/交/差）

```sql
SELECT name FROM t_emp WHERE deptId = 1
UNION ALL
SELECT name FROM t_emp WHERE deptId = 2;
-- 结果 = 集合A ∪ 集合B
-- union 先拼再distinct
-- union all 直接拼 （最常用）

--intersect 交集
--except / minus 差集
```

## 基于派生表的查询

把一条 `select` 的结果，当作一张临时表使用

SQL 没有“变量”，但你需要中间结果。派生表就是 SQL 的“中间变量”。用作结构分解

```sql
-- 先算 → 再查
SELECT *
FROM (
  SELECT deptId, COUNT(*) AS cnt
  FROM t_emp
  GROUP BY deptId
) t
WHERE t.cnt > 5;
```

# 对表中字段的增/删/改（修改表）

# 对数据（元组）的增/删/改

# 空值的处理

# 视图
