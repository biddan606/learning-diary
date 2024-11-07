# Spring Boot testing: Zero to Hero

이 글은 [Spring Boot testing: Zero to Hero by Daniel Garnier-Moiroux](https://www.youtube.com/watch?v=u5foQULTxHM) 유투브 영상을 보고 정리한 글입니다.   

## @autoConfigureMvc

`@autoConfigureMvc`와 `@SpringBootTest`를 이용하면 `mockMvc`를 이용하여 웹 레이어 테스트를 할 수 있습니다.   

![alt text](<image/Spring Boot testing: Zero to Hero/auto-configure-Mvc.png>)

다만 이는 자바로만 이루어져 있습니다.   
자바로만 이루어졌다는 것은 크롬이나 그러한 브라우저가 아닌 자체 브라우저(자바로 된 경량 브라우저)를 이용하여 테스트를 진행합니다.   
그렇기 때문에 실제 렌더링이나 javascript 사용, 사용자와 상호작용은 할 수 없습니다.   

빠르게 웹 레이어의 동작만을 테스트하고 싶은 경우에만 `@autoConfigureMvc`가 적절합니다.

### Springboot 3.4의 간결한 mockMvc

`Springboot 3.4`부터 간결한 방식으로 `mockMvc`를 사용할 수 있습니다.   
`MockMvcTester` 라는 클래스이고 다음과 같이 코드가 간결합니다.

![alt text](<image/Spring Boot testing: Zero to Hero/mockMvcTester.png>)

## @SpringBootTest webEnvironment

`@SpringBootTest`의 `webEnvironment` 설정을 이용하면 실제 요청을 보낼 수 있습니다.   

`webEnvironment`의 기본 값 `mock`을 `RANDOM_PORT`로 변경하면 실제 요청을 보냅니다. 이를 이용하여 디버깅을 찍고 크롬을 통해 살펴볼 수도 있습니다.   

### 또 다른 설정   
**DEFINED_PORT**: 일반적으로 사용하면 안됩니다. 병렬적으로 처리될 경우 예외를 발생시킵니다. `DEFINED_PORT`로 설정된 테스트컨텍스트가 2개가 있다면, 마지막에 실행되는 테스트컨텍스트가 웹서버를 띄우지 못합니다.   
   
**NONE**: 서블릿과 mocking할 필요가 없다면 사용하면 됩니다. mocking하지 않으므로 약간의 성능 향상을 얻을 수 있습니다.
   
    
   
페이지에 `javascript` 코드가 많아 `javascript` 테스트도 함께 이루어져야 한다면 `@SpringBootTest`와 `@autoConfigureMvc`는 적절하지 않습니다.   
셀레니움이나 다른 테스팅 도구를 찾아봐야 합니다.

## TestContext Cache

테스트를 한번에 진행할 때, 여러 개의 테스트 컨텍스트를 띄우는 경우가 있습니다. 이를 줄이면 성능과 메모리 향상을 얻을 수 있습니다.

![alt text](<image/Spring Boot testing: Zero to Hero/slow-bean.png>)

위와 같은 `Configuration`이 있다하고, 테스트 컨텍스트가 여러 번 띄우게 된다면 매번 일정시간의 `sleep`이 발생합니다.   
또한, 그만큼 메모리도 많이 잡아먹게 됩니다.   
이러한 시간을 효율적으로 사용하기 위해서는 같은 테스트컨텍스트를 재활용하면 됩니다.

## 테스트컨텍스트 재활용

테스트 슬라이스별로 공통 테스트 컨텍스트를 만들고 재활용하면 됩니다.   
`@WebMvcTest`, `@DataJpaTest`, `@SpringBootTest` 등의 슬라이스들이 있다고 할 때 아래와 같이 각 슬라이스별 공통 테스트컨텍스트를 사용하면 됩니다.

```java
@SpringBootTest
@Import(CommonTestConfig.class)
@ActiveProfiles("test")
public abstract class BaseTest {
    // 공통 설정 및 유틸리티 메소드
}

public class MyServiceTest1 extends BaseTest {

    @Autowired
    private MyService myService;

    @Test
    void testService1() {
        // 테스트 로직
    }
}
```

이렇게 되면 `springboot`가 자동적으로 같은 환경인 테스트컨텍스트 재활용합니다.(캐시된 테스트컨텍스트를 재활용)

## AssertionJ 확장

상황을 만들고 `AssertionJ`를 확장하지 않았을 때의 코드와 확장했을 때의 코드를 비교해보겠습니다.   
상황은 다음과 같습니다. 매번 같은 패턴의 로그를 찍습니다. 그리고 이 로그가 올바른지를 검사합니다.

### CustomAssertions를 사용하지 않은 테스트 코드

```java
import org.junit.jupiter.api.Test;
import org.assertj.core.api.Assertions;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class LogAssertWithoutCustomAssertionsTest {

    // 로그 메시지 형식을 정의하는 정규 표현식
    private static final Pattern LOG_PATTERN = Pattern.compile(
            "User (?<username>\\w+) has logged in from IP (?<ip>\\d+\\.\\d+\\.\\d+\\.\\d+)");

    /**
     * 로그 메시지를 파싱하여 사용자 이름과 IP 주소를 추출하는 유틸리티 메서드.
     *
     * @param log 실제 로그 메시지
     * @return 매칭된 그룹을 포함하는 Matcher 객체
     */
    private Matcher parseLog(String log) {
        Matcher matcher = LOG_PATTERN.matcher(log);
        Assertions.assertThat(matcher.matches())
                  .withFailMessage("로그 메시지 형식이 올바르지 않습니다: <%s>", log)
                  .isTrue();
        return matcher;
    }

    @Test
    void testValidLogMessage() {
        String log = "User john_doe has logged in from IP 192.168.1.10";
        Matcher matcher = parseLog(log);

        String username = matcher.group("username");
        String ip = matcher.group("ip");

        Assertions.assertThat(username)
                  .withFailMessage("기대하는 사용자 이름: <%s> vs 실제 사용자 이름: <%s>", "john_doe", username)
                  .isEqualTo("john_doe");

        Assertions.assertThat(ip)
                  .withFailMessage("기대하는 IP 주소: <%s> vs 실제 IP 주소: <%s>", "192.168.1.10", ip)
                  .isEqualTo("192.168.1.10");
    }

    @Test
    void testInvalidUsername() {
        String log = "User jane_doe has logged in from IP 10.0.0.5";
        Matcher matcher = parseLog(log);

        String username = matcher.group("username");
        String ip = matcher.group("ip");

        Assertions.assertThat(username)
                  .withFailMessage("기대하는 사용자 이름: <%s> vs 실제 사용자 이름: <%s>", "john_doe", username)
                  .isEqualTo("john_doe"); // 이 부분에서 AssertionError 발생

        Assertions.assertThat(ip)
                  .withFailMessage("기대하는 IP 주소: <%s> vs 실제 IP 주소: <%s>", "10.0.0.5", ip)
                  .isEqualTo("10.0.0.5");
    }

    @Test
    void testInvalidIp() {
        String log = "User john_doe has logged in from IP 127.0.0.1";
        Matcher matcher = parseLog(log);

        String username = matcher.group("username");
        String ip = matcher.group("ip");

        Assertions.assertThat(username)
                  .withFailMessage("기대하는 사용자 이름: <%s> vs 실제 사용자 이름: <%s>", "john_doe", username)
                  .isEqualTo("john_doe");

        Assertions.assertThat(ip)
                  .withFailMessage("기대하는 IP 주소: <%s> vs 실제 IP 주소: <%s>", "192.168.1.10", ip)
                  .isEqualTo("192.168.1.10"); // 이 부분에서 AssertionError 발생
    }

    @Test
    void testInvalidLogFormat() {
        String log = "Invalid log format";

        // 로그 형식이 올바르지 않기 때문에 AssertionError 발생
        parseLog(log);

        // 이후의 어서션은 실행되지 않습니다.
        Matcher matcher = parseLog(log);
        String username = matcher.group("username");
        String ip = matcher.group("ip");

        Assertions.assertThat(username)
                  .isEqualTo("john_doe");

        Assertions.assertThat(ip)
                  .isEqualTo("192.168.1.10");
    }
}
```

`LOG_PATTERN.matcher` 와 같은 동일한 코드가 반복되고 테스트 코드에서 파싱이 이루어져 한 눈에 보기 어렵습니다.

### CustomAssertions 사용

```java
import org.assertj.core.api.AbstractObjectAssert;
import org.assertj.core.api.Assertions;
import java.util.regex.Pattern;

/**
 * 사용자 정의 Assertions 클래스.
 */
public class CustomAssertions extends Assertions {

    /**
     * 특정 형식의 로그 메시지를 검증하기 위한 assertThat 메서드.
     *
     * @param actual 실제 로그 메시지
     * @return LogAssert 인스턴스
     */
    public static LogAssert assertThatLog(String actual) {
        return new LogAssert(actual);
    }

    /**
     * 커스텀 LogAssert 클래스.
     */
    public static class LogAssert extends AbstractObjectAssert<LogAssert, String> {

        // 로그 메시지 형식을 정의하는 정규 표현식
        private static final Pattern LOG_PATTERN = Pattern.compile(
                "User (?<username>\\w+) has logged in from IP (?<ip>\\d+\\.\\d+\\.\\d+\\.\\d+)");

        private String username;
        private String ip;

        /**
         * 생성자에서 로그 메시지를 파싱하여 필드를 초기화합니다.
         *
         * @param actual 실제 로그 메시지
         */
        public LogAssert(String actual) {
            super(actual, LogAssert.class);
            var matcher = LOG_PATTERN.matcher(actual);
            if (!matcher.matches()) {
                throw new AssertionError("로그 메시지 형식이 올바르지 않습니다.");
            }
            this.username = matcher.group("username");
            this.ip = matcher.group("ip");
        }

        /**
         * 사용자 이름을 검증하는 메서드.
         *
         * @param expectedUsername 기대하는 사용자 이름
         * @return 현재 LogAssert 인스턴스
         */
        public LogAssert hasUsername(String expectedUsername) {
            isNotNull();
            Assertions.assertThat(this.username)
                      .withFailMessage("기대하는 사용자 이름: <%s> vs 실제 사용자 이름: <%s>", expectedUsername, this.username)
                      .isEqualTo(expectedUsername);
            return this;
        }

        /**
         * IP 주소를 검증하는 메서드.
         *
         * @param expectedIp 기대하는 IP 주소
         * @return 현재 LogAssert 인스턴스
         */
        public LogAssert hasIp(String expectedIp) {
            isNotNull();
            Assertions.assertThat(this.ip)
                      .withFailMessage("기대하는 IP 주소: <%s> vs 실제 IP 주소: <%s>", expectedIp, this.ip)
                      .isEqualTo(expectedIp);
            return this;
        }
    }
}
```

```java
import org.junit.jupiter.api.Test;

import static CustomAssertions.assertThatLog;

public class LogAssertTest {

    @Test
    void testValidLogMessage() {
        String log = "User john_doe has logged in from IP 192.168.1.10";
        
        assertThatLog(log)
            .hasUsername("john_doe")
            .hasIp("192.168.1.10");
    }

    @Test
    void testInvalidUsername() {
        String log = "User jane_doe has logged in from IP 10.0.0.5";
        
        assertThatLog(log)
            .hasUsername("john_doe") // 이 부분에서 AssertionError 발생
            .hasIp("10.0.0.5");
    }

    @Test
    void testInvalidIp() {
        String log = "User john_doe has logged in from IP 127.0.0.1";
        
        assertThatLog(log)
            .hasUsername("john_doe")
            .hasIp("192.168.1.10"); // 이 부분에서 AssertionError 발생
    }

    @Test
    void testInvalidLogFormat() {
        String log = "Invalid log format";
        
        // 이 부분에서 AssertionError 발생
        assertThatLog(log)
            .hasUsername("john_doe")
            .hasIp("192.168.1.10");
    }
}
```

무엇을 검증하는지 확실하고 코드도 간결해졌습니다.

## 슬라이스 테스트

여러 어노테이션을 이용하여 슬라이스 테스트를 진행할 수 있습니다.   
매번 모든 빈들을 연결하는 것이 아니라 필요한 빈들만 등록하는 것입니다.

### @WebMvcTest

![alt text](<image/Spring Boot testing: Zero to Hero/webmvc.png>)

`@WebMvcTest` 내부에는 `@autoConfigureMvc`도 있고, 웹 영역의 빈들을 등록합니다. `view` 영역의 빈들도 등록하기 때문에 `thymeleaf`를 사용할 경우 `thymeleaf` 테스트도 가능합니다.

### @SpringBootTest

![alt text](<image/Spring Boot testing: Zero to Hero/springboottest.png>)

`view` 영역을 제외한 나머지를 가져옵니다. 모든 빈들을 올려 무겁지만, 그만큼 전체적인 동작을 테스트할 수 있습니다.   
`view` 영역은 등록하지 않으므로, `thymeleaf` 테스트는 불가능합니다.

### @DataJpaTest

![alt text](<image/Spring Boot testing: Zero to Hero/datajpatest.png>)

`Repository` 영역의 빈들만 가져와 가볍게 테스트할 수 있습니다. 다만 `@DataJpaTest`는 기본적으로 인메모리 데이터베이스를 사용하기 떄문에 특정 DB에 종속된 쿼리문을 사용할 경우 직접 변경하여 사용해야 합니다.

## 테스트 컨테이너

[Testcontainers](https://testcontainers.com/)는 `docker`를 활용하여 테스트 환경을 손쉽게 관리하게 해주는 자바 라이브러리입니다.   
원래는 자바 라이브러리였지만, 현재 다른 언어에서도 많이 사용됩니다. `Junit` 등과 통합이 잘되어있어 사용하기 편리합니다.(테스트 세팅이 편한 것이 자바의 큰 장점이 아닌가 싶다)   
   
`Docker`를 사용하기 떄문에 경량화된 외부 인스턴스를 빠르게 생성하고 폐기합니다.   
또한, 높은 유연성과 다양한 인스턴스를 제공해줍니다.

### 사용 방법

![alt text](<image/Spring Boot testing: Zero to Hero/테스트컨텍스트 기본 설정.png>)

위 그림과 같이 간단하게 `postgreSql` 인스턴스를 연결할 수 있습니다.   
기존에는 `@DynamicPropertySource`를 이용하여 수동으로 연결해야 했지만, `SpringBoot 3.1`부터 `@ServiceConnection`를 이용하여 간단하게 연결이 가능합니다.(환경 설정 불필요)

### 주의사항

`테스트 컨테이너`는 `테스트 컨텍스트`마다 새로 생성되고 폐기됩니다.   
이 말은 즉, 5개의 `테스트 컨텍스트`와 3개의 `테스트 컨테이너`가 있다면 5 * 3 = 15개의 `테스트 컨테이너`가 생성되고 폐기된다는 말이다.   
`테스트 컨테이너` 생성에 오랜 시간이 걸리는 것이 있다면 매우 치명적입니다.(테스트 속도 저하)   
이럴 경우 다음과 같이 `테스트 컨테이너`를 싱글톤으로 만들고 사용할 수 있습니다.

![alt text](<image/Spring Boot testing: Zero to Hero/테스트컨테이너문제점해결.png>)

물론 이것도 만능은 아닙니다. 환경이 공유되기 때문에 상태를 가지고 있는 `테스트 컨테이너` 가 있다면 상태를 초기화해주거나 새로 만들어 사용해야 합니다.(싱글톤의 특징)

## 비동기 테스트

![alt text](<image/Spring Boot testing: Zero to Hero/비동기 클래스.png>)

위 코드와 같이 비동기 동작이 있고 이를 테스트하려고 할 때, 보통 `sleep`을 사용합니다.

![alt text](<image/Spring Boot testing: Zero to Hero/sleep테스트.png>)

이와 같은 코드 매번 2초라는 시간을 기다려야 합니다. 테스트가 더 빨리 500ms만에 끝날 수도 있었는데 말이죠.   
이를 해결할 수 있는 아름다운 코드가 있습니다.

![alt text](<image/Spring Boot testing: Zero to Hero/awitility.png>)

![alt text](<image/Spring Boot testing: Zero to Hero/awitility 콘솔.png>)

`Awaitillity`를 사용하면 일정 시간마다 체크하여 통과하면 멈춥니다. 500ms에 끝나면 500ms 언저리에 끝나고, 2s가 걸린다면 2초까지 기다립니다.   
테스트 잘못되어 끝나지 않을 수도 있으므로 `pollDelay` 를 주어 최대 시간을 줄 수 있습니다.   

## mock 깔끔하게 사용하기

![alt text](<image/Spring Boot testing: Zero to Hero/mockito 더러운 코드.png>)

mockito를 사용할 때, 여러 동작을 위와 같이 정의할 수 있습니다. 하지만 깔끔하지 않고 어떤 동작을 원하는지도 알기 쉽지 않습니다.   

![alt text](<image/Spring Boot testing: Zero to Hero/깔끔한 mockito.png>)

mockito의 다양한 메서드를 활용하면 명확학 정의할 수 있습니다.

[mockito](https://site.mockito.org/), [mockito-core](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)에 들어가면 더 다양한 메서드들을 볼 수 있습니다.

## security test

`Spring Security`를 사용하는 경우, 로그인이 되어 있어야 하거나, `csrf`, `OpenID` 등 다양한 제약이 있습니다. 이 때문에 테스트하는 것이 쉽지 않습니다.

`MockMvc`를 사용하면 `csrf()`, `opaqueToken()`, `.oidLogin()` 등을 다양하게 구성할 . 수있습니다.

![alt text](<image/Spring Boot testing: Zero to Hero/security1.png>)

![alt text](<image/Spring Boot testing: Zero to Hero/security2.png>)

`@WithMockUser`, `@WithUserDetailsService` 등은 `@WebMvcTest`도 호환 가능합니다.

## 마무리 약간의 꿀팁

강연자분은 데이터베이스 초기화를 항상 `@BeforeEach`에서 진행한다.   
`@BeforeEach`, `@AfterEach` 큰 차이가 없다고 생각할 수 있지만, 다르다고 한다.   
`@BeforeEach`는 내 테스트 시작 전에 항상 시작이 되지만, `@AfterEach`는 다른 테스트가 항상 돌렸다는 보장이 없다.   
`@BeforeEach`와 `@AfterEach` 중 데이터 초기화를 언제 할지 고민하고 있었는데, `@BeforeEach`에서 수행해야겠다.

## 참조 

[Spring Boot testing: Zero to Hero by Daniel Garnier-Moiroux](https://www.youtube.com/watch?v=u5foQULTxHM)