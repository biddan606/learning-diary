# 테스트 코드에 Fixture 도입기

[Nubble](https://github.com/Nubblee/Nubble-BE) 이라는 서비스를 개발하면서 생긴 문제와 해결방안입니다.   
엔티티에 컬럼을 추가하면서, 테스트 코드가 실패하게 되었고, 모두 컬럼 추가로 인한 실패였습니다. 엔티티 생성을 한 곳에 관리할 필요성이 있게 되었고 `Fixture` 라는 클래스를 추가하게 되었습니다.

## 원인

`User` 엔티티에 `nickname` 컬럼을 추가하였습니다.

```java
@Entity
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_id")
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false)
    private String password;

    // 추가된 컬럼
    @Column(nullable = false)
    private String nickname;

    @Builder
    public User(String username, String password, String nickname) {
        this.username = username;
        this.password = password;
        this.nickname = nickname;
    }
}
```

## 문제 발생

엔티티에 컬럼을 추가한 뒤, 여러 테스트가 실패하였습니다.

![alt text](<image/테스트 코드에 Fixture 도입기/실패한 테스트들.png>)

실패한 테스트들의 에러 로그를 보니 다음과 같았습니다.

![alt text](<image/테스트 코드에 Fixture 도입기/테스트 실패 에러 로그.png>)

테스트를 위해 `User` 엔티티를 저장할 때, `nickname` 값이 null이어서 발생하는 문제였습니다.   

## 해결방안

테스트 성공하기 위해서는 `User` 엔티티 생성시 nickname 필드에 값을 할당해주면 되었습니다. 

```java
// given
User user = User.builder()
        .username("user")
        .password("1234")
        .nickname("kiki") // 테스트 통과하기 위해 필요한 코드
        .build();
userRepository.save(user);
```

하지만 이와 같이 할 경우 약간의 귀찮음이 따릅니다. 실패한 모든 테스트 코드에 반복된 코드를 선언해주어야 합니다.   
현재 프로젝트 초기 단계이기 때문에 큰 불편함이 없지만, 엔티티가 많아질 경우 반복 작업이 많아지고, 테스트 코드 짜기가 싫어질 수도 있습니다.

### Fixture 도입

테스트 코드 엔티티 생성은 매번 똑같은 엔티티를 생성합니다. 하지만 테스트의 독립성을 위해 각 코드마다 생성되어야 합니다.   
`@BeforeEach`에서 일괄 적용할 수도 있지만 테스트 코드의 가독성을 위해 테스트 메서드 내에 짜는 걸 선호합니다.(개인적으로 선호)   

현재 구조를 유지하면서 반복되는 코드를 줄이고자 `UserFixture` 를 도입하게 되었습니다.
```java
public class UserFixture {
    private static final String DEFAULT_USERNAME = "user";
    private static final String DEFAULT_PASSWORD = "1234";
    private static final String DEFAULT_NICKNAME = "kiki";
    private final User.UserBuilder builder;
    
    public static UserFixture aUser() {
        return new UserFixture();
    }
    private UserFixture() {
        this.builder = User.builder()
                .username(DEFAULT_USERNAME)
                .password(DEFAULT_PASSWORD)
                .nickname(DEFAULT_NICKNAME);
    }
    
    public User build() {
        return this.builder.build();
    }
    
    public UserFixture withUsername(String username) {
        this.builder.username(username);
        return this;
    }
    
    public UserFixture withPassword(String password) {
        this.builder.password(password);
        return this;
    }
    
    public UserFixture withNickname(String nickname) {
        this.builder.nickname(nickname);
        return this;
    }
}
```

```java
User user = UserFixture.aUser();
```

이제 엔티티에 변경이 있더라도 `Fixture` 만을 수정하면 되고, 간단한 엔티티를 생성할 경우 1줄에 끝납니다.

## Fixture 코드 설명

#### Fixture를 통한 엔티티 생성시 new 연산자가 아닌 static 메서드를 사용합니다.

```java
public static UserFixture aUser() {
        return new UserFixture();
    }
private UserFixture() {
    this.builder = User.builder()
            .username(DEFAULT_USERNAME)
            .password(DEFAULT_PASSWORD)
            .nickname(DEFAULT_NICKNAME);
}
````

`new` 연산자를 사용하게 되면 `UserFixture`를 생성한다는 느낌을 줄 수 있습니다. static 메서드인 `aUser()`를 통해 `User`를 생성한다는 의도를 명확히 전달합니다.

#### builder는 인스턴스 필드로 선언합니다.

```java
private final User.UserBuilder builder;
```

인스턴스 필드가 아닌 정적 필드일 경우, 병렬 테스트에서 예상치 못한 오류가 발생할 수 있습니다.(각 테스트가 서로 `builder` 필드를 변경 후 생성할 경우)   
이를 방지하고자 `builder`는 인스턴스 필드로 선언합니다.

### 기본값으로 간단하게 객체를 생성하고, with 메서드를 통해 필드값을 변경하도록 합니다.

대부분의 테스트 시에는 default값으로 충분하므로, default 값으로 빠르게 생성하되 필드값 변경시 명시하도록 구현하였습니다.

`lombok` 에서 사용하는 `builder` 와 메서드명을 다르게 하여 명확히 구분하도록 하였습니다.

## Fixture를 도입한 커밋

[Github 커밋 링크](https://github.com/Nubblee/Nubble-BE/commit/e238a41d0e06e60cc29649b00967b40521164a90)