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
