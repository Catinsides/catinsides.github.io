---
title: Leetcode.Database 题解笔记（上）
date: 2016-12-26 11:55:30
tags: MySQL
description: Leetcode是一个在线编程网站，收录了算法，数据库等各种题目。同样有很多优秀的算法解法，能够有效的提高姿势水平。
---
>Leetcode是一个在线编程网站，收录了算法，数据库等各种题目。同样有很多优秀的算法解法，能够有效的提高姿势水平。

闲来无事在leetcode上刷了所有的database题目，一共13道题。题目内容不是简单的CRUD，而是可能很“常用”的查询问题。自我感觉，刷题的过程是一种查缺补漏的过程。除了能够学习新知识，还能够找到自己的不足。这里将每道题的解法及思路记录下来，方便以后查阅。
由于每道题的题目内容较多，仅写下解法与笔记，链接附在题目中。

### 175.[Combine Two Tables](https://leetcode.com/problems/combine-two-tables/)
题目要求为取出Person表中每一个人的FirstName，LastName，City，State信息，无论是否存在地址信息。
解题思路很简单，使用LEFT JOIN即可：
```sql
SELECT Person.FirstName,Person.LastName,Address.City,Address.State
FROM Person LEFT JOIN Address 
ON Person.PersonId = Address.PersonId;
```
这么写可能显得臃肿，可以简化一下：
```sql
SELECT p.FirstName,p.LastName,a.City,a.State
FROM Person p 
LEFT JOIN Address a
USING (PersonId);
```
JOIN USING 简化 JOIN ON 的条件是

* 查询必须等连接；
* 等连接的列必须同名。

### 176.[Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)
题目很好理解，查询Employee表中第二高的薪水值。
解题思路为使用MAX()函数：
```sql
SELECT max(Salary)
FROM Employee
WHERE Salary < (SELECT max(Salary) FROM Employee);
```
提交完后，会报错，因为在第一行后面还要加上 `as SecondHighestSalary`，真是蛋疼的设定啊...

### 177.[Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/)
题目要求为取出表内第N高的薪水值，如果没有则返回null.
这里要编写的为自定义函数，方法如下：
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE M INT;
SET M = N-1;
  RETURN (
      SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT M, 1
  );
END
```
简单记一下LIMIT的用法吧：

* LIMIT 5, 1 //从第6行取出1个
* LIMIT 5, -1 //从第6行至last
* LIMIT 5 // 等价于LIMIT 0, 5 前五行

### 178.[Rank Scores](https://leetcode.com/problems/rank-scores/)
题目内容为按次序排名分数，分数相同的次序相同，分数之间的次序不能跳跃。
解题思路为新建临时表提取去重后的分数，再使用COUNT计数，再与原表中的分数进行笛卡尔积，最后再设置条件筛选：
```sql
SELECT s.Score, COUNT(r.Score) AS Rank
FROM Scores s
     , (
       SELECT DISTINCT Score
         FROM Scores
       ) r
WHERE s.Score <= r.Score
GROUP BY s.Id, s.Score
ORDER BY s.Score DESC;
```
其中，GOURP BY为分组计数。

### 180.[Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)
题目内容为查询出至少出现3次的数字。
解题思路比较巧妙，使用JOIN和新建临时表匹配错位相等的值
```sql
SELECT DISTINCT L1.Num AS ConsecutiveNums
FROM Logs L1
JOIN Logs L2 ON L1.Id + 1 = L2.Id
JOIN Logs L3 ON L1.Id + 2 = L3.Id
WHERE L1.Num = L2.Num AND L1.Num = L3.Num
ORDER BY L1.Num
```
这道题考验思路与技巧，没有什么额外的内容。

### 181.[Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/)
题目内容为查找出比经理收入高的员工。
这道题属于Easy，没啥太大的难点，需要注意的是员工与经理在同一表内，通过Id关联，注意筛选条件即可。
```sql
SELECT a.NAME AS Employee 
FROM Employee a, Employee b 
WHERE a.ManagerId = b.Id AND a.Salary > b.Salary;
```

### 182.[Duplicate Emails](https://leetcode.com/problems/duplicate-emails/)
题目内容为查找表中重复的emails.
这道题也很简单，使用GROUP BY和HAVING 即可。它的作用是代替`WHERE`与合计函数一起使用。
```
SELECT Email FROM Person GROUP BY Email HAVING COUNT(*) > 1;
```

先把前7道题写到这里，后面6道题包含HARD难度，知识点比较多，单独开一篇吧。