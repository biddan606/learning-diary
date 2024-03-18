# 데이터베이스에서 exists()를 사용할 수 있을 땐 count()를 피해라

[Avoid Using COUNT() in SQL When You Could Use EXISTS()](https://blog.jooq.org/avoid-using-count-in-sql-when-you-could-use-exists/)

위 글에서는 exists()로 충분할 땐 count()를 피하라고 한다. 이유는 아래와 같다.   
exists() 쿼리를 데이터베이스에게 보내면, 데이터베이스는 존재하는지만 계산하면 된다. 즉 1행에서 바로 끝날 수도 있다.   
count()로 보내게 된다면 데이터베이스는 항상 모든 열을 확인하여 값을 반환해줄 것이다.(count()를 보낸다는 건 몇개나 존재하는지를 묻는 것)   
자바로 비유하자면 아래와 같다.   
```java
// exists()
if (collection.isEmpty()) {
    return false;
}
return true;

// count()
return collection.size();
```

`collection.isEmpty()` 내부적으로 `collection.size() == 0` 을 비교하여 자바에서는 다른 점이 없다.   
하지만 `size` 라는 값을 캐싱하고 있지 않다면 이야기는 달라진다. `collection.isEmpty()` 는 값이 없는지를 돌아볼테고, `collection.size()` 는 개수를 세어 반환해줄 것이다.   
SQL에서는 모든 요청을 미리 캐싱해둘 수 없으므로 캐싱하고 있지 않을 때처럼 작동한다.

## 결론

`exists()` 요청만으로 충분할 때는 `exsits()` 를 사용하고 절대 `count()` 를 사용하지 말자

## JPA에서는 어떻게 수행할까?

[JPA exists 쿼리 성능 개선](https://jojoldu.tistory.com/516)

`JpaRepository` 내부적으로 limit 1을 사용하여 `exists()` 와 동일한 성능 효과를 누린다.   
`exists()` 를 사용할 수 없는 경우 limit 1 을 사용하면 좋을 것 같다.
