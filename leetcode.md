# Leetcode Solutions

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables/description/)
```
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
```
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
```
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

```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS N
BEGIN
SET N = N-1;
RETURN(SELECT DISTINCT salary FROM employee ORDER BY salary DESC LIMIT 1 OFFSET N);
```

### [178. Rank Scores](https://leetcode.com/problems/rank-scores/description/)
> - rank, group, order, value, index와 같은 alias는 지양하기
> - 특히 윈도우 함수와 함께 쓰게 되면 컬럼인지 함수인지 SQL에 혼란을 준다.

```
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
```
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
```
SELECT employee.name AS Employee
FROM employee
JOIN employee AS manager
ON employee.managerid = manager.id
WHERE 1=1
AND employee.salary > manager.salary
```

### [182. Duplicate Emails](https://leetcode.com/problems/duplicate-emails/description/)
```
SELECT email
FROM person
GROUP BY 1
HAVING COUNT(email) >= 2
```

### [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/description/)
```
SELECT customers.name AS Customers
FROM customers
LEFT JOIN orders
ON orders.customerid = customers.id
WHERE orders.id IS NULL
```
