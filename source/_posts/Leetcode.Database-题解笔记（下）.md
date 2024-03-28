---
title: Leetcode.Database 题解笔记（下）
date: 2016-12-26 15:55:44
tags: MySQL
description: 这篇文章承接上一篇，将Leetcode.Database中的后6道题的解法及笔记记录下来。
---
>Leetcode是一个在线编程网站，收录了算法，数据库等各种题目。同样有很多优秀的算法解法，能够有效的提高姿势水平。

这篇文章承接上一篇，将Leetcode.Database中的后6道题的解法及笔记记录下来。

### 183.[Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)
题目要求为查找出没有订单的客户。
解题思路为使用子查询和NOT IN.
```sql
SELECT Name AS Customers 
FROM Customers c 
WHERE c.id 
NOT IN 
    (SELECT CustomerId FROM Orders);
```
这里需要注意的是要在NAME后添加`AS Customers`.

### 184.[Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)
题目要求为联合员工表和部门表并从中查找出收入最高的员工。
解题思路为使用正确的子查询和过滤条件，
```sql
SELECT d.Name AS Department, e.Name AS Employee, t.Salary 
FROM Employee e 
INNER JOIN 
    (SELECT DepartmentId, MAX(Salary) AS Salary 
     FROM Employee 
     GROUP BY DepartmentId) t
USING (DepartmentId, Salary)
INNER JOIN Department d
ON d.id = t.DepartmentId
```
这里最容易混淆的是SQL语句的执行顺序，简单做个笔记：
SQL语句执行时，每个步骤都会产生一个虚拟表用于下一步骤的输入，虚拟表对外不可用，只会将最终的查询结果返回。如果查询语句中没有某一个子句，则会跳过。
顺序结构如下，小括号内为执行次序：
```sql
(8) SELECT (9)DISTINCT  (11)<Top Num> <select list>
(1) FROM [left_table]
(3) <join_type> JOIN <right_table>
(2) ON <join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(6) WITH <CUBE | RollUP>
(7) HAVING <having_condition>
(10)ORDER BY <order_by_list>
```

### 185.[Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/)
题目要求为查询每个部门前三名收入最高的员工。
解题思路，首先复制Employee表e1，原表为e，在同一部门内选一人与其他员工薪水进行比较，将大于此人的人员数量计数。如果数量为0，则此人为薪水最高者；数量为1，则为第二高者；以此类推……将数量小于3（前三名）的员工，再与部门表做INNER JOIN，最终得出结果。
查询语句如下：
```sql
SELECT d.Name AS Department, e.Name AS Employee, e.Salary AS Salary  
FROM Employee e 
INNER JOIN Department d 
ON e.DepartmentId = d.Id  
WHERE 3 > 
    (
    SELECT COUNT(DISTINCT e1.Salary)  
    FROM Employee e1  
    WHERE e1.Salary > e.Salary  
    AND e1.DepartmentId = e.DepartmentId
    )
```
这道题查询方式很巧妙，需要注意的是将查询方法实践到语句中，同时注意SQL语句的执行顺序。

### 186.[Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)
题目要求为删除重复的Email.
解题思路为复制临时表，取两者交集，删除掉Email相同Id不同的行（取大于小于皆可）。
查询语句如下：
```sql
DELETE p1 
FROM Person p1 
INNER JOIN Person p2
WHERE p1.Email = p2.Email AND p1.Id > p2.Id;
```

### 197.[Rising Temperature](https://leetcode.com/problems/rising-temperature/)
题目要求为查询所有当天温度比前一天高的日期的Id.
解题思路，首先用MySQL内置函数TO_DAYS()将日期转为天数，再使用错位比较（同180）找出温度大于前一天的日期，最后取交集。
查询语句如下：
```sql
SELECT w1.Id 
FROM Weather w1 
INNER JOIN Weather w2
ON TO_DAYS(w1.Date) = TO_DAYS(w2.Date) + 1 
AND w1.Temperature > w2.Temperature;
```

### 262.[Trips and Users](https://leetcode.com/problems/trips-and-users/)
最后一道题，看起来就像压轴题……
题目要求为查询出Oct 1,2013至Oct 3,2013之间，非禁止用户的请求撤销率。
解题思路为根据要求链接两表，设置好过滤条件，再将结果计算后返回。
```sql
SELECT 
    t.Request_at AS `Day`, 
    ROUND(
        SUM(
            IF(t.Status = 'completed', 0, 1)
           ) / COUNT(*),  2) 
        AS `Cancellation Rate` 
FROM Trips t 
INNER JOIN Users u 
ON t.Client_Id = u.Users_Id 
WHERE t.Request_at 
BETWEEN '2013-10-01' AND '2013-10-03' 
AND u.Banned = 'No' AND u.Role = 'client' 
GROUP BY t.Request_at 
ORDER BY t.Request_at;
```
通过之前的解题，理解这段查询语句的逻辑不难。这里仅把前面没提到过的内置函数做下笔记：

* `ROUND()`为四舍五入函数，第二个参数为小数位保留的位数；
* `SUM()`顾名思义为求和函数；
* `IF()`,格式为IF(Condition,A,B)，当Condition为TRUE时，返回A；当Condition为FALSE时，返回B。

### 小结
至此，Leetcode所有Database题目完成（撒花）。总的来看，这些题目锻炼了使用SQL的基本功，同时提供了思考问题的新思路，这在以后的工作中都很有帮助。同时在每页的Discuss中，查看其它网友的解法，能够进一步提高姿势水平。将这些解题的思路整理成两篇文章，也是一个不小的收获。
