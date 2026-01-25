# Advent of SQL 2025 Solutions

Generated on: 1/25/2026, 1:05:50 PM

Total Solved: 25/25

---

## Day 1

> https://solvesql.com/problems/movies-about-love/

```
SELECT title, year, rotten_tomatoes
FROM movies
WHERE LOWER(title) LIKE '%love%'
ORDER BY rotten_tomatoes DESC, year DES
```


---

## Day 2

>https://solvesql.com/problems/good-days-for-a-seoulforest-picnic/
>
> - 날짜 문제를 LIKE로 푸는 방법이 습관이 된 듯 하다.
> - BETWEEN이나 부등호로도 풀이하는 연습을 의식적으로 해야겠다.

```
-- mySQL
SELECT measured_at AS good_day
FROM measurements
WHERE 1=1
AND measured_at LIKE '2022-12%'
AND pm2_5 <= 9
ORDER BY measured_at

-- postgreSQL
SELECT measured_at AS good_day
FROM measurements
WHERE 1=1
AND TO_CHAR(measured_at, 'yyyy-mm') = '2022-12'
AND pm2_5 <= 9
ORDER BY measured_at
```

---

## Day 3
> https://solvesql.com/problems/species-and-mass-of-penguins/

```
SELECT species, body_mass_g
FROM penguins
WHERE 1=1
AND species IS NOT NULL
AND body_mass_g IS NOT NULL
ORDER BY body_mass_g DESC, species
```

---

## Day 4

> https://solvesql.com/problems/whales-of-december/
> 
>  '모든 주문의 매출 합계'의 정의가 모호해서 아래의 계산식 중 고민했고, 모두 실행해보았다.
>> 1. SUM(sum_sales) >= 100
>> 2. HAVING 없이 WHERE 절에서 sum_sales >= 100
>> 3. SUM(sales * quantity) >= 10
>> 4. SUM(sales) >= 100
>
> - 4번 계산식이 제일 아닐 줄 알았는데, 해당 계산식이 정답이었다.
> - 문제에서 좀 더 명확히 정의를 해주었으면 좋았겠지만, 스스로 한번 더 문제에서 구하고자 하는 대상의 계산식을 계획하는 과정의 중요성을 깨닫게 되었다.

```
SELECT records.customer_id
FROM records
LEFT JOIN customer_stats AS stats
ON records.customer_id = stats.customer_id
WHERE 1=1
AND order_date BETWEEN '2020-12-01' AND '2020-12-31'
GROUP BY 1
HAVING SUM(sales) >= 1000
```

---

## Day 5

> https://solvesql.com/problems/count-stamps/

```
SELECT CASE WHEN total_bill >= 25 THEN 2
            WHEN total_bill >= 15 THEN 1
            ELSE 0
            END AS stamp,
       COUNT(total_bill) AS count_bill
FROM tips
GROUP BY 1
ORDER BY stamp
```

---

## Day 6

> https://solvesql.com/problems/dvdrental-vip/
> 
> - 습관적으로 INNER JOIN 대신 일단 모든 데이터를 다 가져오자는 마음으로 LEFT JOIN을 써왔던 습관을 알아차렸다.
> - 무작정 LEFT JOIN을 쓰기보다는 테이블 간의 관계를 명확히 이해하고 '나는 INNER JOIN한 데이터만 필요하다'는 확신과 계획을 가지고 쿼리를 짜야겠다.

```
-- mySQL
SELECT customer.customer_id
FROM rental
LEFT JOIN customer
ON rental.customer_id = customer.customer_id
WHERE 1=1
AND customer.active = 1
GROUP BY 1
HAVING COUNT(rental_id) >= 35
LIMIT 100

-- postgreSQL
SELECT customer.customer_id
FROM rental
LEFT JOIN customer
ON rental.customer_id = customer.customer_id
WHERE 1=1
AND customer.active = '1'
GROUP BY 1
HAVING COUNT(rental_id) >= 35
LIMIT 100
```

---

## Day 7

> https://solvesql.com/problems/bad-finddust-days-in-a-row/

```
-- mySQL
WITH tool AS
(
SELECT *, 
       pm10 AS today,
       LAG(pm10, 1) OVER (ORDER BY measured_at) AS yesterday,
       LAG(pm10, 2) OVER (ORDER BY measured_at) AS twodaysago
FROM measurements
WHERE YEAR(measured_at) = 2022
)

SELECT measured_at AS date_alert
FROM tool
WHERE 1=1
AND twodaysago < yesterday
AND yesterday < today
AND today >= 30
ORDER BY measured_at ASC



-- postgreSQL
WITH tool AS
(
SELECT *, 
       pm10 AS today,
       LAG(pm10, 1) OVER (ORDER BY measured_at) AS yesterday,
       LAG(pm10, 2) OVER (ORDER BY measured_at) AS twodaysago
FROM measurements
WHERE 1=1
AND TO_CHAR(measured_at, 'YYYY') = '2022'
)

SELECT measured_at AS date_alert
FROM tool
WHERE 1=1
AND twodaysago < yesterday
AND yesterday < today
AND today >= 30
ORDER BY measured_at ASC
```

---

## Day 8

> https://solvesql.com/problems/wines-for-friends/

```
SELECT *
FROM wines
WHERE 1=1
AND color = 'white'
AND quality >= 7
AND density > (SELECT AVG(density) FROM wines)
AND residual_sugar > (SELECT AVG(residual_sugar) FROM wines)
AND pH < (SELECT AVG(pH) FROM wines WHERE color = 'white')
AND citric_acid > (SELECT AVG(citric_acid) FROM wines WHERE color = 'white')
```

---

## Day 9

> https://solvesql.com/problems/volleyball-players-in-two-consecutive-olympics/
>
> 1. 쿼리 구조 설계
> - 궁극적으로 메인 쿼리에서 조회해야 하는 것은 '두 대회 연속으로 출전한 기록이 있는' 배구 선수를 찾는 것이라고 판단
> - '2016년까지 대한민국 국가대표팀에 소속되어 여자 배구 종목'까지는 CTE에 한번에 넣어버리는 것이 깔끔하다고 생각
>
> 2. 고민 포인트: '두 대회 연속 출전' 계산식 수립
> - 모든 경기가 올림픽 경기라는 사실을 알지 못했다. 많은 경기 중에 올림픽 경기가 있는 줄 알았는데, 테이블을 조회해보니 각 올림픽별 경기가 있었다.
> - 위의 사실을 모르고 처음에는 '경기ID로 두 대회 연속 출전을 식별하나?' 생각해서 ROW_NUMBER + LAG로 접근해보았는데 원하는 결과가 출력되지 않았다.
> - '설마 모든 경기 = 올림픽, 그래서 4년?' 하는 생각으로 두 대회 연속 출전 조건을 걸었더니 정답 처리되었다.

```
WITH findtheplayer AS
(
SELECT athletes.id AS id,
       athletes.name AS name,
       teams.team AS team,
       events.event AS event,
       games.year AS year,
       events.id AS event_id,
       LAG(year, 1) OVER (PARTITION BY athletes.id ORDER BY events.id) AS year_2

FROM records
JOIN teams
ON records.team_id = teams.id
JOIN games
ON records.game_id = games.id
JOIN events
ON records.event_id = events.id
JOIN athletes
ON records.athlete_id = athletes.id

WHERE 1=1
AND events.event = 'Volleyball Women''s Volleyball'
AND teams.team = 'KOR'
)


SELECT DISTINCT id, name
FROM findtheplayer
WHERE year - year_2 = 4
LIMIT 100
```

---

## Day 10

> https://solvesql.com/problems/volleyball-players-with-medals/
>
> - 테이블 확인해보니 메달 2개 이상 받은 선수 없었다. 따라서 GROUP_CONCAT과 GROUP BY문 없이 SELECT medals만 해도 정답 처리가 되기는 한다.
> - postgreSQL로 풀 경우 STRING_AGG 함수 사용!
``` STRING_AGG(records.medal, ',') AS medals ```

```
-- mySQL
SELECT athletes.id AS id,
       athletes.name AS name,
       GROUP_CONCAT(records.medal SEPARATOR ',') AS medals
FROM records
JOIN teams
ON records.team_id = teams.id
JOIN games
ON records.game_id = games.id
JOIN events
ON records.event_id = events.id
JOIN athletes
ON records.athlete_id = athletes.id

WHERE 1=1
AND events.event = 'Volleyball Women''s Volleyball'
AND teams.team = 'KOR'
AND games.year <= 2016
AND records.medal IS NOT NULL

GROUP BY 1,2
```

---

## Day 11

> https://solvesql.com/problems/revenue-weekday-weekend/
>
> - postgreSQL로 풀 경우 큰 따옴표를 작은 따옴표로 조정 필요!

```
-- mysql
SELECT CASE WHEN day IN ('Sat', 'Sun') THEN 'weekend'
            ELSE 'weekday'
            END AS week,
       SUM(total_bill) AS sales
FROM tips
GROUP BY 1
ORDER BY 2 DESC
```

---

## Day 12

> https://solvesql.com/problems/critic-scores-by-genre-and-year/
> 
> - 처음에는 COALESCE에서 'critic_score IS NOT NULL' AND 조건까지 붙이는 방법으로 접근했다.
> - 그러나 2011년부터 2015년까지를 COALESCE 구문을 작성하다보니 WHERE 조건으로 빼는 것이 더 깔끔하겠다고 판단하여 그렇게 진행했다.
> - 이렇게 작성한 쿼리 효율도 더 좋다. 왜냐하면 CASE 안에 조건을 넣으면, DB는 전체 테이블을 스캔한 뒤 각 행에서 CASE 조건을 연산해야 하기 때문!

```
SELECT genres.name AS genre,
       ROUND(AVG(COALESCE(CASE WHEN year = 2011 THEN games.critic_score END, NULL)),2) AS score_2011,
       ROUND(AVG(COALESCE(CASE WHEN year = 2012 THEN games.critic_score END, NULL)),2) AS score_2012,
       ROUND(AVG(COALESCE(CASE WHEN year = 2013 THEN games.critic_score END, NULL)),2) AS score_2013,
       ROUND(AVG(COALESCE(CASE WHEN year = 2014 THEN games.critic_score END, NULL)),2) AS score_2014,
       ROUND(AVG(COALESCE(CASE WHEN year = 2015 THEN games.critic_score END, NULL)),2) AS score_2015
FROM games
JOIN genres
ON games.genre_id = genres.genre_id
WHERE critic_score IS NOT NULL
GROUP BY 1
ORDER BY 1
```

---

## Day 13

> https://solvesql.com/problems/top-revenue-actors/
>
> [풀이 실패 요인]
>
>> 1. GROUP BY
>> - 처음에는 한치의 의심도 없이 GROUP BY 1,2 로 접근했다. (이름과 성)
>> - 그러나 계속 틀리는 것을 보고 (물론 그것이 유일한 이유는 아니었으나) actor 테이블을 다시 확인해보았고 actor_id를 발견해서 GROUP BY를 수정해주었다.
>>
>> 2. FK 찾기
>> - rental, payment 테이블을 조인할 때 처음에는 customer_id를 활용했다.
>> - 급한 마음으로 풀다보니 customer_id가 가장 먼저 눈에 들어왔던 것 같다.
>> - id 라고 하니 당연히 FK겠지 하는 안일한 마음으로 조인했고, 더 수정할 부분이 없어 보였는데 자꾸 정답이 아니라고 했다.
>> - 답답해 하던 중 원택님이 FK를 다시 확인해보라고 하셔서 테이블을 다시 확인해보았고 rental_id를 발견했다.
>> - 생각해보니 고객이 중복해서 대여를 한 경우도 있을 터이고 실제로 그런 경우가 확인되었다. 경솔했다! FK 찾기를 더욱 신중히 하자.


```
SELECT actor.first_name AS first_name,
       actor.last_name AS last_name,
       SUM(payment.amount) AS total_revenue
      
FROM film_actor
JOIN actor ON film_actor.actor_id = actor.actor_id
JOIN inventory ON film_actor.film_id = inventory.film_id
JOIN rental ON inventory.inventory_id = rental.inventory_id
JOIN payment ON rental.rental_id = payment.rental_id

GROUP BY actor.actor_id
ORDER BY 3 DESC
LIMIT 5
```

---

## Day 14

> https://solvesql.com/problems/monthly-author-candidates/
>
> [주요 사고 포인트]
> - 한 연도에 두 번 이상 수상한 작가는 없을까?
> - 이에 간단히 쿼리 짜서 확인해보았는데 모두 수상은 일 년에 한 번씩만 한 것으로 확인
> - 따라서 그냥 author로 그룹바이 해서 COUNT(year) 해도 괜찮겠다고 판단
>
> [문제 해석 오류 포인트]
>
> - 처음에는 "해당 작가 작품들의 평균 리뷰 수"를 소설/비소설을 불문한 평균 리뷰 수인 줄 알아서 WHERE 조건 없이 쿼리를 짰다.
> - 그러나 코드를 작성할수록 '소설의 평균 리뷰 수가 그렇게 상징적인건가?', '수상한 작가들의 전체 장르 평균 리뷰 수랑 소설의 전체 평균 리뷰 수를 왜 비교하지?' 생각이 들었다.
> - WHERE문을 수정했고 정답처리되었다.

```
SELECT author
FROM books
WHERE genre = 'Fiction'
GROUP BY author
HAVING 1=1
AND COUNT(year) >= 2
AND AVG(user_rating) >= 4.5
AND AVG(reviews) >= (SELECT AVG(reviews) FROM books WHERE genre = 'Fiction')
ORDER BY 1
```

---

## Day 15

> https://solvesql.com/problems/find-movies-by-korean-artists/
>
> - 무난했으나 어제의 레슨런 (FK를 신중히 찾기)을 톡톡히 활용할 수 있었던 문제였다: FK는 artist_id가 아니라 artwork_id

```
SELECT artists.name AS artist,
       artworks.title AS title
FROM artworks
JOIN artworks_artists
ON artworks.artwork_id = artworks_artists.artwork_id
JOIN artists
ON artworks_artists.artist_id = artists.artist_id

WHERE 1=1
AND artists.nationality = 'Korean'
AND artworks.classification LIKE 'Film%'
```

---

## Day 16

> https://solvesql.com/problems/friendship-between-early-users/

```
WITH base AS
(
SELECT user_a_id, 
       user_b_id,
       SUM(user_a_id + user_b_id) AS id_sum
FROM edges
GROUP BY 1, 2
),

ranked AS 
(
SELECT *,
       PERCENT_RANK() OVER (ORDER BY id_sum ASC) * 100 AS top
FROM base
)

SELECT user_a_id, 
       user_b_id,
       id_sum
FROM ranked
WHERE top <= 0.1
```

---

## Day 17

> https://solvesql.com/problems/first-order-category/
>
> [시행착오: 첫 구매로 주문된 건수 계산식]
> - 정답: count(distinct order_id)
> - 착각: quantity
>
> - quantity로 접근해서 풀이했을 때 계속 오답처리 되어서 테이블을 다시 찬찬히 확인해보았다.
> - records 말고도 customer_stats 테이블이 하나 더 있다는 사실을 뒤늦게 깨달았다.
> - 조인해보면서 문제의 의도와 계산식을 이해하게 되었다.

```
SELECT category, 
       sub_category,
       COUNT(DISTINCT order_id) AS cnt_orders
FROM records
JOIN customer_stats AS stats
ON records.customer_id = stats.customer_id
WHERE 1=1
AND records.order_date = stats.first_order_date
GROUP BY 1,2
ORDER BY 3 DESC
```

---

## Day 18

> https://solvesql.com/problems/influencer-marketing-candidates/
>
> - CTE들을 어떻게 구성할지 생각하는 데 시간이 오래 걸렸다. 작게 분리하고 난 후에는 그나마 수월했지만 아래 시행착오를 겪는 시간이 길었다.
>
> [시행착오]
>
>> 1. 메인 쿼리
>> - 처음에 메인 쿼리에 100명의 친구를 가진 후보 조건을 WHERE문으로 넣어서 접근했다. 
>> - 그랬더니 CTE에서 edges 테이블을 계속 조인하게 되는 비효율이 발생했다.
>> - thetable로 CTE를 분리했고 조건문을 중복해서 작성해야 했던 문제가 해결되었다.
>>
>>  2. 후보 친구의 친구 수 구하기
>> - 처음에는 유저별 친구 수를 구하는(friend_cnt) CTE를 만들 생각을 못했다. 
>> - 'count한걸 어떻게 sum 하지?' 하는 고민을 하다가 결국 분리해보았는데, 분리했더니 수월해졌다.
>> - 'friend_cnt를 참조해서 SUM을 하면 되는구나!'를 깨달았을 때 희열이 있었다.

```
WITH thetable AS (
    SELECT user_a_id AS user_id, user_b_id AS friend_id FROM edges
    UNION ALL
    SELECT user_b_id AS user_id, user_a_id AS friend_id FROM edges
),
friend_cnt AS (
    SELECT
        user_id,
        COUNT(friend_id) AS friends
    FROM thetable
    GROUP BY user_id
),
aboutme AS (
    SELECT
        user_id,
        friends
    FROM friend_cnt
    WHERE friends >= 100
),
aboutthem AS (
    SELECT
        thetable.user_id,
        SUM(friend_cnt.friends) AS friends_of_friends
    FROM thetable
    JOIN aboutme
      ON thetable.user_id = aboutme.user_id
    JOIN friend_cnt 
      ON thetable.friend_id = friend_cnt.user_id
    GROUP BY 1
)
SELECT
    aboutme.user_id,
    aboutme.friends,
    aboutthem.friends_of_friends,
    ROUND(aboutthem.friends_of_friends / aboutme.friends, 2) AS ratio
FROM aboutme
JOIN aboutthem
  ON aboutme.user_id = aboutthem.user_id
ORDER BY ratio DESC
LIMIT 5
```

---

## Day 19

> https://solvesql.com/problems/yearly-net-sales/
> 
> - postgreSQL로 푼다면 BOOLEAN과 INTEGER 간 암묵적 형변환을 허용하지 않기 때문에:
``` WHERE is_returned = FALSE ```
``` WHERE is_returned::integer = 0 ```


```
--mySQL

SELECT EXTRACT(YEAR from purchased_at) AS year,
       SUM(total_price - discount_amount) AS net_sales
FROM transactions
WHERE is_returned = 0
GROUP BY 1
ORDER BY 1


--postgreSQL

SELECT TO_CHAR(purchased_at, 'yyyy') AS year,
       SUM(total_price - discount_amount) AS net_sales
FROM transactions
WHERE is_returned = FALSE
GROUP BY 1
ORDER BY 1

```

---

## Day 20

> https://solvesql.com/problems/yearly-shipping-usage/
>
> [문제를 풀며 중요하다고 생각한 두 가지 포인트]
> - shipping_method 값이 있다는 것은 그 자체로 온라인 배송이다 (테이블도 확인 완료)
> - 그러나 반품은 온라인/오프라인 구매 모두에서 발생할 수 있으므로  is_returned = 1 일 때 스탠다드 배송에 1을 더해줄 때는 is_online_order = 1이라는 조건을 추가해줘야 한다. 
```

WITH base AS
(
SELECT purchased_at,
       shipping_method,
       CASE WHEN is_returned = 1 AND is_online_order = 1 THEN 1 ELSE 0 END AS returned_cnt
FROM transactions
)

SELECT EXTRACT(YEAR from purchased_at) AS year,
       SUM(COALESCE(CASE WHEN shipping_method = 'Standard' THEN 1 END, 0)) + SUM(returned_cnt) AS standard,
       SUM(COALESCE(CASE WHEN shipping_method = 'Express' THEN 1 END, 0)) AS express,
       SUM(COALESCE(CASE WHEN shipping_method = 'Overnight' THEN 1 END, 0)) AS overnight
FROM base
GROUP BY 1
ORDER BY 1
```

---

## Day 21

> https://solvesql.com/problems/ab-testing-buckets-1/
>
> - customer_id가 UNIQUE한 값이 아니므로 GROUP BY 필요!

```
SELECT customer_id,
       CASE WHEN customer_id % 10 = 0 THEN 'A'
            ELSE 'B' END AS bucket
FROM transactions
GROUP BY 1
ORDER BY 1
```

---

## Day 22

> https://solvesql.com/problems/cumulative-orders/
>
> 1. 오해 포인트
> - 주문건수가 물량 수의 SUM의 개념인 줄 알았는데 단순 횟수 COUNT 였다.
> 
> 2. 학습 포인트
> - TO_CHAR(calendar.order_date, 'Day') AS weekday 로 풀게 되면 'Thrusday '와 같이 공백이 생긴다.
> - 따라서 'Day'가 아닌 'FMDay'로 풀어야 한다.

```
-- postgreSQL
WITH daily AS 
(
SELECT TO_CHAR(calendar.order_date, 'yyyy-mm-dd') AS order_date,
       TO_CHAR(calendar.order_date, 'FMDay') AS weekday,
       COALESCE(COUNT(transactions.quantity), 0) AS num_orders_today
FROM generate_series(DATE '2023-10-31', DATE '2023-12-31', INTERVAL '1 day') AS calendar(order_date)
LEFT JOIN transactions
ON transactions.purchased_at::date = calendar.order_date
AND transactions.is_online_order IS TRUE
GROUP BY 1,2
)


SELECT order_date,
       weekday,
       num_orders_today,
       num_orders_today ::numeric + LAG(num_orders_today, 1, 0) OVER (ORDER BY order_date) ::numeric AS num_orders_from_yesterday
FROM daily
ORDER BY 1
OFFSET 1
```

---

## Day 23

> https://solvesql.com/problems/ab-testing-buckets-2/

> - SELECT level에서 집계함수 AVG와 CNT를 함께 쓸 수 없으므로 CTE로 구분 필요!

```
WITH user_group AS 
(
SELECT CASE WHEN customer_id % 10 = 0 THEN 'A'
            ELSE 'B' END AS bucket,
       customer_id,
       COUNT(transaction_id) AS order_cnt,
       SUM(total_price) AS revenue
FROM transactions
WHERE is_returned = 0
GROUP BY 1, 2
)
SELECT bucket,
       COUNT(customer_id) AS user_count,
       ROUND(AVG(order_cnt), 2) AS avg_orders,
       ROUND(AVG(revenue), 2) AS avg_revenue
FROM user_group
GROUP BY 1
ORDER BY 1

```

---

## Day 24

> https://solvesql.com/problems/vip-of-cities/
>
> - 이미 grain 이 완성된 테이블은 다시 더 낮은 grain 테이블과 JOIN하지 않는다.
>
> - 처음에 letsrank에서 city_id를 SELECT하지 않고, 메인 쿼리에서 base랑 조인을 진행했다. (base에 city_id가 있기 때문에)
> - 단순히 'city_id 값을 가져와야지' 생각했지 둘을 customer_id로 조인했을 때 도시별 ROW를 집계한 값이 와해될 것을 예상하지 못했다.
> - letsrank에서 city_id SELECT 하고 메인 쿼리에서 조인하지 않음으로써 문제를 해결했다.

```
WITH base AS (
SELECT city_id,
       customer_id,
       SUM(total_price - discount_amount) AS total_spent
FROM transactions
WHERE is_returned = 0
GROUP BY 1,2
)
,
letsrank AS (
SELECT city_id,
       customer_id,
       total_spent,
       ROW_NUMBER() OVER (PARTITION BY city_id ORDER BY total_spent DESC) AS ranking
FROM base
)

SELECT city_id,
       customer_id,
       total_spent
FROM letsrank
WHERE ranking = 1
```

---

## Day 25

> https://solvesql.com/problems/laugh-of-santa-claus/
>
> ho ho ho, merry chistmas! 🎄

```
SELECT 'Ho Ho Ho'
```

---

