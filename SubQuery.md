# 서브 쿼리

관계형 데이터베이스의 쿼리 안에 들어간 쿼리를 서브 쿼리라고 부른다. SELECT, FROM, WHERE 등 다양한 절에 들어갈 수 있습니다.   
서브 쿼리의 종류에 대해 알아보고 언제 사용하면 좋고 안좋은지 알아보겠습니다.   

## 서브 쿼리의 종류

서브 쿼리의 종류를 살펴볼 때 사용할 에제 테이블입니다.

첫 번째 테이블 : galleries

| id  | city     |
|-----|----------|
| 1   | London   |
| 2   | New York |
| 3   | Munich   |

두 번째 테이블 : paintings

| id  | name            | gallery_id | price |
|-----|-----------------|-------------|-------|
| 1   | Patterns        | 3           | 5000  |
| 2   | Ringer          | 1           | 4500  |
| 3   | Gift            | 1           | 3200  |
| 4   | Violin Lessons  | 2           | 6700  |
| 5   | Curiosity       | 2           | 9800  |

세 번째 테이블 : sales_agents

| id  | last_name | first_name | gallery_id | agency_fee |
|-----|-----------|------------|------------|------------|
| 1   | Brown     | Denis      | 2          | 2250       |
| 2   | White     | Kate       | 3          | 3120       |
| 3   | Black     | Sarah      | 2          | 1640       |
| 4   | Smith     | Helen      | 1          | 4500       |
| 5   | Stewart   | Tom        | 3          | 2130       |

| id  | product_id | sale_date  | amount |
|-----|------------|------------|--------|
| 1   | 1          | 2023-01-15 | 3      |
| 2   | 1          | 2023-02-20 | 2      |
| 3   | 2          | 2022-11-30 | 1      |
| 4   | 2          | 2023-03-05 | 5      |
| 5   | 3          | 2023-01-25 | 4      |
| 6   | 4          | 2023-02-10 | 2      |
| 7   | 4          | 2023-03-12 | 1      |
| 8   | 5          | 2022-12-30 | 3      |
| 9   | 5          | 2023-01-15 | 6      |

네 번째 테이블 : managers

| id  | gallery_id |
|-----|------------|
| 1   | 2          |
| 2   | 3          |
| 4   | 1          |

### 스칼라 서브쿼리

하나의 행과 하나의 열을 반환하는 서브쿼리입니다. 주로 SELECT, WHERE, HAVING 절에서 사용될 수 있으며, 하나의 행렬을 예상하여 연산자를 붙이기 때문에 여러 행렬을 반환할 경우 예외를 발생시킬 수 있습니다.

#### JOIN으로 변경 예시

예제: 각 그림이 속한 갤러리의 도시 이름을 조회

**스칼라 서브쿼리**

```sql
SELECT 
  p.name, 
  (SELECT g.city FROM galleries g WHERE g.id = p.gallery_id) AS city
FROM paintings p;
```

**JOIN**

```sql
SELECT 
  p.name, 
  g.city
FROM paintings p
JOIN galleries g ON p.gallery_id = g.id;
```

예제: 각 갤러리의 평균 에이전시 수수료 조회

**스칼라 서브쿼리**

```sql
SELECT 
  g.city,
  (SELECT AVG(sa.agency_fee) FROM sales_agents sa WHERE sa.gallery_id = g.id) AS avg_agency_fee
FROM galleries g;
```

**JOIN**

```sql
SELECT 
  g.city, 
  AVG(sa.agency_fee) AS avg_agency_fee
FROM galleries g
JOIN sales_agents sa ON g.id = sa.gallery_id
GROUP BY g.city;
```

#### JOIN으로 변경 불가능 예시

가격이 5000 이상인 각 그림의 2023년 이후 총 판매 수량

```sql
SELECT 
  p.name, 
  (SELECT SUM(s.amount) 
   FROM sales s 
   WHERE s.product_id = p.id AND s.sale_date >= '2023-01-01') AS total_sales_since_2023
FROM paintings p
WHERE p.price > 5000;
```

2023년 이후의 판매 수

```sql
SELECT 
  p.name, 
  (SELECT COUNT(*) 
   FROM sales s 
   WHERE s.product_id = p.id AND s.date >= '2023-01-01') AS sales_count_since_2023
FROM products p;
```

메인 쿼리 각 행에 대해 별도의 집계가 필요한 경우 JOIN 대신 스칼라 서브쿼리를 사용하는 것이 바람직할 수 있습니다.

### multiple-row 서브쿼리

하나 이상의 행을 반환하는 서브쿼리입니다. 주로 IN, ANY, ALL, EXISTS 와 함께 사용합니다.   

#### JOIN으로 변경 예시

예제: 뉴욕에 위치한 갤러리 모든 그림의 이름 조회

**스칼라 서브쿼리**

```sql
SELECT name
FROM paintings
WHERE gallery_id IN (SELECT id FROM galleries WHERE city = 'New York');
```

**JOIN**

```sql
SELECT p.name
FROM paintings p
JOIN galleries g ON p.gallery_id = g.id
WHERE g.city = 'New York';
```

예제: 그림이 있는 갤러리 도시 조회

**스칼라 서브쿼리**

```sql
SELECT city
FROM galleries g
WHERE EXISTS (SELECT 1 FROM paintings p WHERE p.gallery_id = g.id);
```

**JOIN**

```sql
SELECT DISTINCT g.city
FROM galleries g
JOIN paintings p ON g.id = p.gallery_id;
```

DISTINCT를 이용하여 도시 이름 중복 제거

#### JOIN으로 변경 불가능 예시

예제: gallery_id가 2인 갤러리에서 가격이 가장 높은 상위 3개의 그림을 조회

```sql
SELECT *
FROM paintings p
WHERE p.id IN (
    SELECT id
    FROM (
        SELECT id
        FROM paintings
        WHERE gallery_id = 2
        ORDER BY price DESC
        LIMIT 3
    ) AS top_paintings
);
```

내부 서브쿼리에서 GROUP BY와 HAVING 절을 사용하여 집계된 결과를 반환합니다. 이러한 집계 결과가 JOIN으로는 자연스럽지 않을 수 있습니다.   
또한, 집계된 결과를 필터링하는 로직을 JOIN으로 구현하기 쉽지 않습니다.

Multiple-row 서브쿼리는 특정 조건을 만족하는 여러 행을 반환하기 때문에, 이러한 서브쿼리를 JOIN으로 변환하는 것이 자연스럽지 않거나 불가능한 경우가 있습니다. 특히, LIMIT, 복잡한 집계 함수, 연관 서브쿼리 등 동적 조건을 포함하는 경우에 이러한 서브쿼리를 JOIN으로 변환하는 것은 어려울 수 있습니다.

## 서브쿼리와 JOIN 비교

### 서브쿼리 장단점

**장점**
- 모듈화: 쿼리의 특정 부분을 독립적으로 작성할 수 있습니다.
- 복잡한 조건 처리: 메인 쿼리에서 처리하기 어려운 복잡한 조건을 쉽게 처리할 수 있습니다.
- 중첩된 데이터 처리: 필터링을 통해 중첩된 데이터를 최소화하여 메모리의 부하가 적습니다.

**단점**
- 성능 저하: 서브 쿼리가 각 행마다 실행된다면 매우 큰 성능 저하를 일으킬 수 있습니다.
- 복잡성 증가: 서브 쿼리가 중첩되면 쿼리가 복잡해질 수 있습니다. (1개의 서브쿼리만 존재하더라도 JOIN의 가독성이 높은 경우가 많습니다)
- 비효율적인 집계 처리: 복잡한 집계 처리를 하다보면 비효율적인 쿼리가 될 수 있습니다.

### JOIN 장단점

**장점**
- 직관적 관계 표현: 테이블 간의 관계를 명확히 표현할 수 있습니다.
- 효율적인 집계 처리: 서브 쿼리처럼 여러번 실행하지 않아 효율적입니다.

**단점**
- 메모리 사용량 증가: 대규모 데이터셋을 조인하면 메모리 사용량이 급격히 증가할 수 있습니다.
- 디스크 I/O 부하: 많은 데이터를 디스크에서 읽고 쓰기 때문에 디스크 I/O 부하가 발생할 수 있습니다.

| 항목               | 서브쿼리                                                 | JOIN                                                    |
|--------------------|----------------------------------------------------------|---------------------------------------------------------|
| **기본 개념**      | 쿼리 내에 포함된 또 다른 쿼리                            | 여러 테이블을 결합하여 데이터를 조회                    |
| **사용 용도**      | 복잡한 조건 처리, 중첩된 데이터 필터링, 존재 여부 확인   | 테이블 간의 관계 표현, 효율적인 데이터 결합              |
| **성능**           | 각 행마다 실행되어 성능 저하 가능                        | 데이터베이스 엔진이 최적화하여 빠르게 실행               |
| **가독성**         | 복잡한 쿼리를 논리적으로 나눔                            | 테이블 간의 관계를 명확히 표현                          |
| **최적화**         | 인덱스를 적절히 사용하지 않으면 어려움                   | 다양한 인덱스를 사용하여 성능 최적화 가능               |
| **복잡성**         | 중첩된 서브쿼리는 복잡성이 증가할 수 있음                | 복잡한 조인 쿼리는 최적화가 어려울 수 있음              |
| **비효율적인 집계**| 집계 연산이 포함된 경우 비효율적일 수 있음               | 복잡한 집계 연산을 효율적으로 처리                      |
| **메모리 사용량**  | 상대적으로 적음                                          | 대규모 데이터셋을 조인하면 메모리 사용량이 증가할 수 있음|


## 결론

- 서브쿼리는 복잡한 조건 처리와 중첩된 데이터 필터링에 유용하지만, 성능 저하와 복잡성 증가의 단점이 있습니다.

- JOIN은 테이블 간의 관계를 명확히 표현하고, 효율적인 집계 연산을 처리할 수 있지만, 대규모 데이터셋에서 메모리 사용량과 디스크 I/O 부하가 증가할 수 있습니다. 


## 참조

- [서브 쿼리의 종류에는 무엇이 있을까?](https://kimsyoung.tistory.com/entry/%EC%84%9C%EB%B8%8C-%EC%BF%BC%EB%A6%AC%EC%9D%98-%EC%A2%85%EB%A5%98%EC%97%90%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4-%EC%9E%88%EC%9D%84%EA%B9%8C)

- [SUBQUERY 와 JOIN 의 차이 (上)](https://kimsyoung.tistory.com/entry/SUBQUERY-%EC%99%80-JOIN-%EC%9D%98-%EC%B0%A8%EC%9D%B4-%E4%B8%8A)
- [SUBQUERY 와 JOIN 의 차이 (下)](https://kimsyoung.tistory.com/entry/SUBQUERY-%EC%99%80-JOIN-%EC%9D%98-%EC%B0%A8%EC%9D%B4-%E4%B8%8B)