# 해시맵

key-value pair들을 저장하는 자료구조, 같은 키를 가지는 pair는 최대 1개까지만 존재합니다.   
매우 빠른 검색, 삽입, 삭제를 가능하게 해줍니다.

## 비슷한 구조로는 무엇이 있을까?

- TreeMap: iterate시에 정렬된 키를 반환해줍니다.
- LinkedHashMap: 삽입 순서를 보장해줍니다. LRU(Least Recently Used) 알고리즘 사용시, 유용합니다. 재삽입시 키 순서를 마지막으로 이동시키려면 `accessOrder = true` 설정을 해주어야 합니다.

## 언제 사용할까?

- 캐싱: 해시 함수를 잘 설계했을 경우, 탐색시 O(1)이 걸리므로 빠르게 탐색할 수 있습니다.
- 중복 검사: 해시맵에 저장하고, 데이터 추가 시에 해시맵에 이미 해당 키가 존재하는지 확인함으로써, 중복 검사를 할 수 있습니다. 


## HashSet과 무엇이 다를까?

![alt text](<image/해시셋 add함수.png>)

`HashSet.add()` 연산 시 내부적으로는 해시맵을 사용하고 있습니다. 해시 셋 == 키 값만 존재하는 해시맵이라고 볼 수 있습니다.

## 해시맵의 동작 방식

### 삽입

1. key 값을 기준으로 해시값을 얻습니다.
2. 해시값을 해시맵의 배열 크기로 나눈 나머지 값을 구한 뒤, 해시맵의 내부 배열[나머지 값] 위치에 key-value를 넣습니다.
3. 이미 배열의 인덱스에 pair가 존재한다면, hash collision을 해결합니다.
    - java에서는 separate chaining 방식으로 해결합니다.

내부적으로 3/4 이상의 인덱스(버킷)을 사용하고 있으면 hash collision가 일어날 확률이 높다고 판단합니다.   
hash collision가 높아지면 성능이 떨어지기 때문에 배열 확장을 한 뒤, 존재하는 pair들을 재삽입하는 과정이 일어납니다.

### 추출

1. key 값을 해시맵에 맞는 해시값으로 변환합니다.
2. 얻은 해시값의 위치(버킷)에서 자신의 해시값과 같은 해시값을 찾습니다.
    - 같은 해시값이 없다면, false를 반환합니다.
3. 같은 해시값이 존재한다면, `equals()` 를 비교합니다.
    - `equals()` 가 true라면 value를 반환, false라면 null을 반환합니다.


### 포함 확인

내부적으로 `getNode()` 를 불러 Node를 반환하면 true, null이라면 false를 반환합니다. 

### 삭제

1. 키값으로 Node를 추출합니다.
2. 자바에서는 hash collision 해결방식으로 separate chaining을 사용하고 있어, 앞뒤 값이 존재할 수 있습니다. 존재할 경우 삭제할 노드를 제외한 앞뒤 노드들을 연결시킵니다.

## 파이썬 딕셔너리와 비교

![alt text](<image/파이썬 딕셔너리와 자바 해시맵 비교.png>)

파이썬의 딕셔너리는 자바의 해시맵과 다르게 해시 충돌 해결 방법을 Open addressing을 사용합니다.   
Open addressing은 해시값이 충돌하면 해당 버킷(인덱스)에 넣는 것이 아니라 새로운 버킷을 탐색합니다.   
이러한 방식 때문에 문제가 발생할 수 있습니다.(기존 버킷을 차지하고 있던 pair가 사라질 경우, 새로운 버킷을 탐색한 pair들을 찾지 못할 수 있음)   
파이썬의 딕셔너리는 이러한 문제점을 삭제시 더미 데이터를 넣는 방식으로 해결하고, 더미 데이터가 많이 쌓일 경우 shrink를 진행합니다.

### 차이

- 해시 충돌 해결 방법
- 더미 데이터 존재
- shrink

## 리스트와 해시맵 또는 해시셋을 어떻게 선택하면 좋을까?

리스트가 단순하기 때문에 iteration시에 더 빠르고 메모리도 효율적으로 사용합니다. 또한, 해시를 사용하려면 추가적으로 `equals()` 와 `hashCode()` 도 설정해주어야 합니다.   
해시를 이용할 필요가 없는 경우 리스트가 바람직합니다.

## 해시맵 사용시 주의해야 할 점

```java
public class Location {
    int x;
    int y;

    public Location(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

위와 같은 Location 클래스를 해시맵에 넣을 경우 발생할 수 있는 문제들에 대해 알아보겠습니다.

### equals and hashCode 

`HashSet.add(new Location(1, 1))` 2번 진행하게 되면 해시셋의 사이즈가 2가 됩니다. 해시셋은 중복을 제거한다고 했는데 왜 그럴까요?   
이유는 해시셋을 저장할 때 사용하는 `hashCode` 와 `equals` 를 재정의하지 않았기 때문입니다. 재정의하지 않을 경우 `Object` 클래스의 설정을 사용하게 하여, `hashCode` 와 `equals` 모두 메모리 주소를 기준으로 비교합니다.   
Hash 관련 자료구조를 사용할 때는 `hashCode` 와 `equals` 재정의를 해주어야 합니다.

### 필드 값 추가

해시셋에 `Location` 객체를 추가하고 나서, `hashCode` 와 `equals` 재정의 해줄 경우(필드에 z값을 추가하여 재정의가 필요한 경우 등) 문제가 발생할 수 있습니다.   
해시셋을 저장할 때 사용한 해시값과 변경된 해시값이 달라져 해시셋에서 정확한 버킷을 찾을 수 없게 됩니다.

### 값 변경

해시셋에 `Location` 객체를 추가하고 나서, `Location`의 필드를 수정하게 된 경우에도 이전 해시값과 달라져 찾을 수 없게 됩니다.(예를 들어, `location.x = 100`)   
이 문제를 해결하기 위해, 해시셋에 들어가는 객체의 필드들은 final로 선언해주는 것이 좋습니다.

## 참조

[맵(map)과 해시 테이블(hash table) 핵심만 모아보기! - 쉬운 코드](https://www.youtube.com/watch?v=ZBu_slSH5Sk)

[셋(set)과 셋의 대표 구현체인 해시 셋(hash set)의 핵심 - 쉬운 코드](https://www.youtube.com/watch?v=IkImFugfFQk)

[값이 중복되는 객체를 제거할 목적으로 hash set을 쓰려면? - 쉬운 코드](https://www.youtube.com/watch?v=Dmo3sG-ZFTw)

[Hash Table은 프로그래머의 기본기 - 포프TV](https://www.youtube.com/watch?v=S7vni1hdsZE)
