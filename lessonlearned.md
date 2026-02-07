### 1. Window function과 Group by 충돌
> [550. Game Play Analysis IV](https://leetcode.com/problems/game-play-analysis-iv/description/)

```sql
SELECT player_id,
       MIN(event_date) AS initial,
       LEAD(event_date, 1) OVER (ORDER BY event_date) AS following
FROM activity
GROUP BY 1
```

> [위의 쿼리문 오류]
> - SQL 실행 순서상 GROUP BY로 데이터를 먼저 뭉친 후 그 결과 위에 윈도우 함수 LEAD가 실행됨
> - GROUP BY 1을 하는 순간, 한 플레이어당 행에는 최초 로그인 날짜만 남게 됨
> - 이미 플레이어별로 event_date가 한 행만 남은 상태에서 LEAD를 SELECT하면 조회하게 되는 정보는 'A 유저의 다음 방문일'이 아니라 '다른 유저의 최초 방문일'이 됨
> - 즉, 한 사람의 연속 방문을 체크하는 게 아니라 다른 사람의 방문일 데이터를 가져오게 되는 것

```sql
WITH numbering_table AS (
SELECT player_id,
        event_date,
        ROW_NUMBER() OVER (PARTITION BY player_id ORDER BY event_date) AS numbering,
        LEAD(event_date) OVER (PARTITION BY player_id ORDER BY event_date) AS following_date
FROM activity
)

SELECT ROUND(AVG(CASE WHEN DATEDIFF(following_date, event_date) = 1 THEN 1 ELSE 0 END), 2
) AS fraction
FROM numbering_table
WHERE numbering = 1
```

> - 우선 GROUP BY로 데이터를 압축하지 않고, 윈도우 함수를 활용해 유저별 전체 접속 이력의 순서와 익일 접속일을 확보
> - 준비된 데이터에서 최초 접속일(numbering = 1) 데이터만 추출한 뒤, event_date와 following_date의 간격이 1일인 케이스를 집계하여 연속 접속 여부를 판별
> - numbering = 1 조건이 없으면 '최초 접속일 기준의 재방문율'이 아니라, 단순히 전체 기간 중 이틀 연속 방문한 모든 케이스를 집계하게 됨



### 2. 연속된 구간 분석하기
> [601. Human Traffic of Stadium](https://leetcode.com/problems/human-traffic-of-stadium/description/)

```sql
WITH base AS
(
SELECT id,
       visit_date,
       people,
       CASE WHEN people >= 100 THEN 1 ELSE 0 END AS indicator
FROM stadium
)
,
calculate AS
(
SELECT *,
       SUM(CASE WHEN indicator = 1 THEN 1 ELSE 0 END) OVER (ORDER BY id) * indicator AS consecutive
FROM base
)

SELECT id, visit_date, people
FROM calculate
WHERE consecutive >= 3
```

> [위의 쿼리문 오류]
> - 연속이 끊겼을 때(인원이 100명 미만일 때) 합계가 다시 0이나 1부터 시작되지 않고 직전 숫자를 유지하며 누적됨
> - '연속 3일'이 아니라 '지금까지 총 3일'을 구하게 됨: SUM(CASE WHEN indicator = 1 THEN 1 ELSE 0 END) OVER (ORDER BY id)


```sql
WITH base AS
(
SELECT id,
       visit_date,
       people,
       ROW_NUMBER () OVER (ORDER BY visit_date) AS number_cl,
       id - ROW_NUMBER () OVER (ORDER BY visit_date) AS group_cl
FROM stadium
WHERE people >= 100
)
,
finalize AS
(
SELECT *,
       COUNT(id) OVER (PARTITION BY group_cl) AS count_cl
FROM base
)

SELECT id,
       visit_date,
       people
FROM finalize
WHERE count_cl >= 3
ORDER BY visit_date
```


> -연속된 숫자라면 id가 1 커질 때, ROW_NUMBER()도 똑같이 1 커진다.
> - 따라서 두 값의 차이인 group_cl는 연속되는 동안 일정한 상수로 유지된다.
> - 이 상수값을 통해 연속된 날짜들을 하나의 그룹으로 묶고, 그 그룹에 포함된 값을 카운팅하여 3일 연속되었는지 여부를 판단할 수 있다.
