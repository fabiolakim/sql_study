# Leetcode Solutions

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables/description/)
```sql
SELECT person.firstName,
       person.lastName,
       address.city,
       address.state
FROM person
LEFT JOIN address
ON person.personid = address.personid
```

### [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)
> 1. 나의 풀이: DENSE_RANK와 CASE WHEN + MAX로 두 번째로 큰 값을 하나만 조회하기
```sql
WITH findthesecond AS
(
SELECT salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS ranking
FROM employee
)

SELECT MAX(CASE WHEN ranking = 2 THEN salary END) AS SecondHighestSalary
FROM findthesecond
```
> 2. 쿼리 효율 고려: DISTINCT + LIMIT + OFFSET으로 두 번째 큰 값 조회하기
> - DISTINCT + OFFSET 1로 두 번째 값이 존재하지 않으면 결과가 0행이 된다.
> - 이를 스칼라 서브쿼리로 감싸면 0행 대신 NULL을 반환할 수 있다.
> - DENSE_RANK를 쓰면 모든 행의 순위를 스캔해야 하지만, LIMIT + OFFSET은 가장 큰 값을 건너뛰고 다음 값만 불러오면 종료
```sql
SELECT
(
SELECT DISTINCT salary AS SecondHighestSalary
FROM employee
ORDER BY salary DESC
LIMIT 1
OFFSET 1
)
```

### [177. Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/description/)
> CREATE FUNCTION 함수 구문을 처음 접했다.
> - 데이터베이스 내에서 반복적인 작업을 자동화하고, SQL 쿼리에서 효율성을 극대화하는 함수
> - 일반 SQL과 달리 stored function(이번 문제), stored procedure의 경우 세미콜론 문법이 중요하다.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS N
BEGIN
SET N = N-1;
RETURN(SELECT DISTINCT salary FROM employee ORDER BY salary DESC LIMIT 1 OFFSET N);
```

### [178. Rank Scores](https://leetcode.com/problems/rank-scores/description/)
> - rank, group, order, value, index와 같은 alias는 지양하기
> - 특히 윈도우 함수와 함께 쓰게 되면 컬럼인지 함수인지 SQL에 혼란을 준다.

```sql
WITH letsrank AS
(
    SELECT id,
           score,
           DENSE_RANK() OVER (ORDER BY score DESC) AS 'rank'
    FROM scores
)
SELECT score, 'rank'
FROM letsrank
```

### [180. Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)
> SQL은 파이썬처럼 a=b=c 비교 연산자를 연달아 사용할 수 없다!
```sql
WITH base AS
(
SELECT id,
       num,
       LAG(num) OVER (ORDER BY id) AS atras,
       LEAD(num) OVER (ORDER BY id) AS frente
FROM logs
)

SELECT DISTINCT num AS ConsecutiveNums
FROM base
WHERE num = atras AND num = frente
```

### [181. Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/description/)
```sql
SELECT employee.name AS Employee
FROM employee
JOIN employee AS manager
ON employee.managerid = manager.id
WHERE 1=1
AND employee.salary > manager.salary
```

### [182. Duplicate Emails](https://leetcode.com/problems/duplicate-emails/description/)
```sql
SELECT email
FROM person
GROUP BY 1
HAVING COUNT(email) >= 2
```

### [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/description/)
```sql
SELECT customers.name AS Customers
FROM customers
LEFT JOIN orders
ON orders.customerid = customers.id
WHERE orders.id IS NULL
```

### [184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/description/)
> CTE에서 RANK 함수를 이용하여 메인쿼리에서 RANK=1로 찾기
```sql
WITH base AS
(
SELECT department.name AS Department,
       employee.name AS Employee,
       employee.salary AS Salary,
       RANK() OVER (PARTITION BY employee.departmentid ORDER BY employee.salary DESC) AS ranking
FROM employee
JOIN department
ON employee.departmentid = department.id
)

SELECT Department, Employee, Salary
FROM base
WHERE ranking =1
```

> 부서별 MAX(salary)를 찾는 WHERE 조건문 활용
```sql
SELECT department.name AS Department,
       employee.name AS Employee,
       employee.salary AS Salary
FROM employee
JOIN department
ON employee.departmentid = department.id
WHERE (departmentid, employee.salary) IN (SELECT departmentid, MAX(salary) FROM employee GROUP BY 1)
```

### [185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/description/)
```sql
WITH base AS
(
    SELECT name, 
           salary, 
           departmentid,
           DENSE_RANK() OVER (PARTITION BY departmentid ORDER BY salary DESC) AS ranking
    FROM employee
)

SELECT department.name AS Department,
       base.name AS Employee,
       base.salary AS Salary
FROM base
JOIN department
ON base.departmentid = department.id
WHERE base.ranking <= 3
```

### [196. Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)
> 처음 작성한 쿼리 (오답)
> Error: - You can't specify target table 'person' for update in FROM clause
```sql
DELETE FROM person
WHERE id NOT IN (SELECT MIN(id) AS min_id FROM person GROUP BY email)
```

> DELETE, UPDATE와 같이 테이블을 수정하는 함수에서는 수정 중인 원본 데이터를 직접 읽으면 결과가 꼬일 수 있으니, 임시 테이블을 만들어 참조하게 함으로써 데이터의 충돌을 방지해야 한다.
```sql
DELETE FROM person
WHERE id NOT IN (SELECT min_id FROM (SELECT MIN(id) AS min_id FROM person GROUP BY email) AS base)
```

### [197. Rising Temperature](https://leetcode.com/problems/rising-temperature/description/)
> 주어진 테이블의 날짜가 연속적이라는 보장이 없으므로, temperature 뿐만 아니라 date에 대해서도 전일인지 확인이 필요하다.
```sql
WITH base AS
(
SELECT *,
       LAG(recorddate, 1) OVER (ORDER BY recorddate) AS yesterday_date,
       LAG(temperature, 1) OVER (ORDER BY recorddate) AS yesterday_temp
FROM weather
)

SELECT Id
FROM base
WHERE 1=1
AND DATEDIFF(recorddate, yesterday_date) = 1
AND yesterday_temp < temperature
```

### [262. Trips and Users](https://leetcode.com/problems/trips-and-users/description/)
> - 처음 범한 오류: JOIN + ON + OR로 테이블을 조인하고 users_id에 driver, client 모두 포섭되니 users.banned = 'No'만 하면 될 것이라 생각
> - 레슨런: 참조 테이블의 한 컬럼이 대상 테이블의 서로 다른 두 컬럼과 관계를 맺을 때는, 각각 별도로 셀프 조인하여 각 조건이 동시에 충족되는지 검증해야 한다
```sql
WITH base AS
(
SELECT trips.id,
       trips.client_id,
       trips.driver_id,
       trips.status,
       trips.request_at,
       client.users_id,
       client.banned AS c_banned,
       driver.banned AS d_banned
FROM trips 
JOIN users AS client
ON trips.client_id = client.users_id
JOIN users AS driver
ON trips.driver_id = driver.users_id
WHERE 1=1
AND client.banned = 'No'
AND driver.banned = 'No'
AND DATE(trips.request_at) IN ('2013-10-01', '2013-10-02', '2013-10-03')
)

SELECT request_at AS Day,
       ROUND(SUM(CASE WHEN status = 'completed' THEN 0 ELSE 1 END) / COUNT(id), 2) AS 'Cancellation Rate'
FROM base
GROUP BY 1
```

### [511. Game Play Analysis I](https://leetcode.com/problems/game-play-analysis-i/description/)
```sql
SELECT player_id, MIN(event_date) AS first_login
FROM activity
GROUP BY 1
```


### [570. Managers with at Least 5 Direct Reports](https://leetcode.com/problems/managers-with-at-least-5-direct-reports/description/)
```sql
WITH base AS
(
SELECT employee.id AS emp_id,
       manager.id AS man_id,
       manager.name AS man_name
FROM employee
JOIN employee AS manager
ON employee.managerid = manager.id
)

SELECT man_name AS name
FROM base
GROUP BY man_id
HAVING COUNT(emp_id) >= 5
```


### [585. Investments in 2016](https://leetcode.com/problems/investments-in-2016/description/)
```sql
WITH base AS
(
SELECT pid, tiv_2015, tiv_2016
FROM insurance
WHERE 1=1
AND tiv_2015 IN (SELECT tiv_2015 FROM insurance GROUP BY 1 HAVING COUNT(*) > 1)
AND (lat,lon) IN (SELECT lat,lon FROM insurance GROUP BY 1,2 HAVING COUNT(*) = 1)
)

SELECT ROUND(SUM(tiv_2016),2) AS tiv_2016
FROM base
```
