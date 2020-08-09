# 175. 组合两个表 #

表1: Person

	+-------------+---------+
	| 列名         | 类型     |
	+-------------+---------+
	| PersonId    | int     |
	| FirstName   | varchar |
	| LastName    | varchar |
	+-------------+---------+
	PersonId 是上表主键

表2: Address

	+-------------+---------+
	| 列名         | 类型    |
	+-------------+---------+
	| AddressId   | int     |
	| PersonId    | int     |
	| City        | varchar |
	| State       | varchar |
	+-------------+---------+
	AddressId 是上表主键

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

	FirstName, LastName, City, State

----------

	SELECT FirstName, LastName, City, State
	FROM Person 
	LEFT JOIN Address
	ON Person.PersonId = Address.PersonId

# 176. 第二高的薪水 #

编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。

	+----+--------+
	| Id | Salary |
	+----+--------+
	| 1  | 100    |
	| 2  | 200    |
	| 3  | 300    |
	+----+--------+

例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。

	+---------------------+
	| SecondHighestSalary |
	+---------------------+
	| 200                 |
	+---------------------+


----------

	SELECT
	    IFNULL(
	      (SELECT DISTINCT Salary
	       FROM Employee
	       ORDER BY Salary DESC
	        LIMIT 1， 1),
	    NULL) AS SecondHighestSalary

# 177. 第N高的薪水 #

编写一个 SQL 查询，获取 Employee 表中第 n 高的薪水（Salary）。

	+----+--------+
	| Id | Salary |
	+----+--------+
	| 1  | 100    |
	| 2  | 200    |
	| 3  | 300    |
	+----+--------+

例如上述 Employee 表，n = 2 时，应返回第二高的薪水 200。如果不存在第 n 高的薪水，那么查询应返回 null。

	+------------------------+
	| getNthHighestSalary(2) |
	+------------------------+
	| 200                    |
	+------------------------+


----------

	CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
		BEGIN
		  RETURN (
		      # Write your MySQL query statement below.
		      SELECT DISTINCT a.Salary 
		      FROM Employee a
		      WHERE N-1 = (
	              SELECT COUNT(DISTINCT b.Salary) 
	              FROM Employee b 
	              WHERE b.Salary > a.Salary)
		  );
		END

# 178. 分数排名 #

编写一个 SQL 查询来实现分数排名。

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

	+----+-------+
	| Id | Score |
	+----+-------+
	| 1  | 3.50  |
	| 2  | 3.65  |
	| 3  | 4.00  |
	| 4  | 3.85  |
	| 5  | 4.00  |
	| 6  | 3.65  |
	+----+-------+

例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：

	+-------+------+
	| Score | Rank |
	+-------+------+
	| 4.00  | 1    |
	| 4.00  | 1    |
	| 3.85  | 2    |
	| 3.65  | 3    |
	| 3.65  | 3    |
	| 3.50  | 4    |
	+-------+------+

重要提示：对于 MySQL 解决方案，如果要转义用作列名的保留字，可以在关键字之前和之后使用撇号。例如 `Rank`

	SElECT a.Score AS Score, 
	(SElECT COUNT(DISTINCT b.Score) FROM Scores b WHERE b.Score >= a.Score) AS 'Rank'
	FROM Scores a
	ORDER BY a.Score DESC

# 180. 连续出现的数字 #

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

	+----+-----+
	| Id | Num |
	+----+-----+
	| 1  |  1  |
	| 2  |  1  |
	| 3  |  1  |
	| 4  |  2  |
	| 5  |  1  |
	| 6  |  2  |
	| 7  |  2  |
	+----+-----+

例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。

	+-----------------+
	| ConsecutiveNums |
	+-----------------+
	| 1               |
	+-----------------+


----------

	SELECT DISTINCT l1.Num AS ConsecutiveNums
	FROM Logs l1, Logs l2, Logs l3
	WHERE l1.Id = l2.Id - 1 
	    AND l2.Id = l3.Id - 1
	    AND l1.Num = l2.Num
	    AND l1.Num = l3.Num

# 181. 超过经理收入的员工 #

Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

	+----+-------+--------+-----------+
	| Id | Name  | Salary | ManagerId |
	+----+-------+--------+-----------+
	| 1  | Joe   | 70000  | 3         |
	| 2  | Henry | 80000  | 4         |
	| 3  | Sam   | 60000  | NULL      |
	| 4  | Max   | 90000  | NULL      |
	+----+-------+--------+-----------+

给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

	+----------+
	| Employee |
	+----------+
	| Joe      |
	+----------+

	SELECT a.Name AS Employee
	FROM Employee a, Employee b
	WHERE a.Salary > b.Salary
	    AND a.ManagerId = B.Id

# 182. 查找重复的电子邮箱 #

编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。

	+----+---------+
	| Id | Email   |
	+----+---------+
	| 1  | a@b.com |
	| 2  | c@d.com |
	| 3  | a@b.com |
	+----+---------+

根据以上输入，你的查询应返回以下结果：

	+---------+
	| Email   |
	+---------+
	| a@b.com |
	+---------+


----------

	SELECT DISTINCT Email
	FROM Person
	GROUP BY Email
	HAVING COUNT(*) > 1

# 183. 从不订购的客户 #

某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

Customers 表：

	+----+-------+
	| Id | Name  |
	+----+-------+
	| 1  | Joe   |
	| 2  | Henry |
	| 3  | Sam   |
	| 4  | Max   |
	+----+-------+

Orders 表：

	+----+------------+
	| Id | CustomerId |
	+----+------------+
	| 1  | 3          |
	| 2  | 1          |
	+----+------------+

例如给定上述表格，你的查询应返回：

	+-----------+
	| Customers |
	+-----------+
	| Henry     |
	| Max       |
	+-----------+


----------

	select customers.name as 'Customers'
	from customers
	where customers.id not in
	(
	    select customerid from orders
	);

# 184. 部门工资最高的员工 #

Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

	+----+-------+--------+--------------+
	| Id | Name  | Salary | DepartmentId |
	+----+-------+--------+--------------+
	| 1  | Joe   | 70000  | 1            |
	| 2  | Jim   | 90000  | 1            |
	| 3  | Henry | 80000  | 2            |
	| 4  | Sam   | 60000  | 2            |
	| 5  | Max   | 90000  | 1            |
	+----+-------+--------+--------------+

Department 表包含公司所有部门的信息。

	+----+----------+
	| Id | Name     |
	+----+----------+
	| 1  | IT       |
	| 2  | Sales    |
	+----+----------+

编写一个 SQL 查询，找出每个部门工资最高的员工。对于上述表，您的 SQL 查询应返回以下行（行的顺序无关紧要）。

	+------------+----------+--------+
	| Department | Employee | Salary |
	+------------+----------+--------+
	| IT         | Max      | 90000  |
	| IT         | Jim      | 90000  |
	| Sales      | Henry    | 80000  |
	+------------+----------+--------+


----------


	SELECT Department.Name AS 'Department', Employee.Name AS 'Employee', Salary
	FROM Employee JOIN Department
	On Employee.DepartmentId = Department.id
	WHERE (Employee.DepartmentId, Salary) IN 
	(
	    SELECT DepartmentId, MAX(Salary)
	    FROM Employee
	    GROUP BY DepartmentId
	)

# 196. 删除重复的电子邮箱 #

编写一个 SQL 查询，来删除 Person 表中所有重复的电子邮箱，重复的邮箱里只保留 Id 最小 的那个。

	+----+------------------+
	| Id | Email            |
	+----+------------------+
	| 1  | john@example.com |
	| 2  | bob@example.com  |
	| 3  | john@example.com |
	+----+------------------+
	Id 是这个表的主键。

例如，在运行你的查询语句之后，上面的 Person 表应返回以下几行:

	+----+------------------+
	| Id | Email            |
	+----+------------------+
	| 1  | john@example.com |
	| 2  | bob@example.com  |
	+----+------------------+


----------

	DELETE a
	FROM Person a, Person b
	WHERE a.Email = b.Email AND a.Id > b.Id

# 197. 上升的温度 #

给定一个 Weather 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。

	+---------+------------------+------------------+
	| Id(INT) | RecordDate(DATE) | Temperature(INT) |
	+---------+------------------+------------------+
	|       1 |       2015-01-01 |               10 |
	|       2 |       2015-01-02 |               25 |
	|       3 |       2015-01-03 |               20 |
	|       4 |       2015-01-04 |               30 |
	+---------+------------------+------------------+

例如，根据上述给定的 Weather 表格，返回如下 Id:

	+----+
	| Id |
	+----+
	|  2 |
	|  4 |
	+----+

----------

	SELECT a.Id AS Id
	FROM WEATHER a, WEATHER b
	WHERE DATEDIFF(a.RecordDate, b.RecordDate) = 1
	    AND a.Temperature > b.Temperature

# 595. 大的国家 #

这里有张 World 表

	+-----------------+------------+------------+--------------+---------------+
	| name            | continent  | area       | population   | gdp           |
	+-----------------+------------+------------+--------------+---------------+
	| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
	| Albania         | Europe     | 28748      | 2831741      | 12960000      |
	| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
	| Andorra         | Europe     | 468        | 78115        | 3712000       |
	| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
	+-----------------+------------+------------+--------------+---------------+

如果一个国家的面积超过300万平方公里，或者人口超过2500万，那么这个国家就是大国家。

编写一个SQL查询，输出表中所有大国家的名称、人口和面积。

例如，根据上表，我们应该输出:

	+--------------+-------------+--------------+
	| name         | population  | area         |
	+--------------+-------------+--------------+
	| Afghanistan  | 25500100    | 652230       |
	| Algeria      | 37100000    | 2381741      |
	+--------------+-------------+--------------+


----------

	SELECT name, population, area
	FROM World
	WHERE area > 3000000 OR population > 25000000

# 596. 超过5名学生的课 #

有一个courses 表 ，有: student (学生) 和 class (课程)。
请列出所有超过或等于5名学生的课。
例如,表:

	+---------+------------+
	| student | class      |
	+---------+------------+
	| A       | Math       |
	| B       | English    |
	| C       | Math       |
	| D       | Biology    |
	| E       | Math       |
	| F       | Computer   |
	| G       | Math       |
	| H       | Math       |
	| I       | Math       |
	+---------+------------+

应该输出:

	+---------+
	| class   |
	+---------+
	| Math    |
	+---------+


----------

	SELECT class
	FROM courses
	GROUP BY class
	HAVING COUNT(DISTINCT student) >= 5

# 627. 交换工资 #

给定一个 salary 表，如下所示，有 m = 男性 和 f = 女性 的值。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求只使用一个更新（Update）语句，并且没有中间的临时表。

注意，您必只能写一个 Update 语句，请不要编写任何 Select 语句。

	| id | name | sex | salary |
	|----|------|-----|--------|
	| 1  | A    | m   | 2500   |
	| 2  | B    | f   | 1500   |
	| 3  | C    | m   | 5500   |
	| 4  | D    | f   | 500    |


----------


	UPDATE salary
	SET
	    sex = CASE sex
	        WHEN 'm' THEN 'f'
	        ELSE 'm'
	    END;