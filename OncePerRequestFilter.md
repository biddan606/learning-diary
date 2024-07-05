# OncePerRequestFilter

1번의 http 요청에 1번만 실행하는 필터

## 정리하게 된 이유
Spring Security를 공부하던 중, OncePerRequestFilter 사용하는 것을 보았습니다.   
1번의 http 요청에 1번만 실행하는 필터라고 하는데, 일반 Filter도 그렇지 않나하여 정리해봅니다.

## 1번의 요청이 필터를 여러 번 호출하는 경우

일반적으로 Get, Post 요청 같은 경우, 요청이 들어오면 1번의 filter만 거치게 됩니다. 하지만 특수한 경우 여러 번 호출될 수도 있습니다.   
filter의 위치부터 보자면, filter는 내장 톰캣과 디스패처 서블릿 사이에 위치합니다.   
응답 값이 반환하지 않고 이 경로를 다시 들어온다면 filter를 여러 번 호출할 수 있습니다.
요청의 반환 값이 foward, error, redirect 요청이라면 다시 filter를 거치게 됩니다.(redirect의 경우, 응답값을 반환하고 클라이언트가 재요청하는 것이지만 확실한 테스트를 위해 포함시켰습니다)

## 컨트롤러 테스트 코드

foward, error, redirect 하는 경우를 만들어 테스트해보겠습니다.
또한, filter를 등록할 수 있는 방법이 2가지이기 때문에 각각 테스트해보겠습니다.

```java
@RestController
@RequestMapping("/filter")
@Slf4j
public class FilterTestController {

    @GetMapping("/test-forward")
    public void testForward(HttpServletRequest request, HttpServletResponse response)
            throws IOException, ServletException {
        log.info("/test-forward 호출");
        request.getRequestDispatcher("/filter/forwarded").forward(request, response);
    }

    @GetMapping("/forwarded")
    public String forwarded() {
        log.info("/forwarded 호출");
        return "Forwarded";
    }

    @GetMapping("/test-redirect")
    public void testRedirect(HttpServletResponse response) throws IOException {
        log.info("/test-redirect 호출");
        response.sendRedirect("/filter/redirected");
    }

    @GetMapping("/redirected")
    public String redirected() {
        log.info("/redirected 호출");
        return "Redirected";
    }

    @GetMapping("/test-error")
    public String errorTest() {
        log.info("/error-test 호출");
        throw new RuntimeException("Test Error");
    }
}
```

## @Component를 통해 filter를 등록 예제

filter와 OncePerRequestFilter 구현체를 @Component를 통해 필터로 등록하여 테스트해보겠습니다.

```java
@Component
@Slf4j
public class MyFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        log.info("MyFilter - 처리중인 URL= {}", httpRequest.getRequestURI());

        chain.doFilter(request, response);
    }
}
```

```java
@Component
@Slf4j
public class MyOncePerRequestFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        log.info("MyOncePerRequestFilter - 처리중인 URL= {}", request.getRequestURI());

        filterChain.doFilter(request, response);
    }
}
```

### @Component 테스트 결과

**FORWARD**

```
GET http://localhost:8080/filter/test-forward

2024-07-05T16:34:53.969+09:00  INFO 54361 --- [nio-8080-exec-4] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/test-forward
2024-07-05T16:34:53.979+09:00  INFO 54361 --- [nio-8080-exec-4] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/test-forward
2024-07-05T16:34:53.986+09:00  INFO 54361 --- [nio-8080-exec-4] c.e.s.filter.FilterTestController        : /test-forward 호출
2024-07-05T16:34:53.987+09:00  INFO 54361 --- [nio-8080-exec-4] c.e.s.filter.FilterTestController        : /forwarded 호출
```

**REDIRECT**

```
GET http://localhost:8080/filter/test-redirect

2024-07-05T16:36:33.149+09:00  INFO 54361 --- [nio-8080-exec-7] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/test-redirect
2024-07-05T16:36:33.150+09:00  INFO 54361 --- [nio-8080-exec-7] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/test-redirect
2024-07-05T16:36:33.158+09:00  INFO 54361 --- [nio-8080-exec-7] c.e.s.filter.FilterTestController        : /test-redirect 호출
2024-07-05T16:36:33.198+09:00  INFO 54361 --- [nio-8080-exec-8] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/redirected
2024-07-05T16:36:33.199+09:00  INFO 54361 --- [nio-8080-exec-8] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/redirected
2024-07-05T16:36:33.200+09:00  INFO 54361 --- [nio-8080-exec-8] c.e.s.filter.FilterTestController        : /redirected 호출
```

**ERROR**

```
GET http://localhost:8080/filter/test-error

2024-07-05T16:37:06.644+09:00  INFO 54361 --- [nio-8080-exec-1] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/test-error
2024-07-05T16:37:06.657+09:00  INFO 54361 --- [nio-8080-exec-1] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/test-error
2024-07-05T16:37:06.666+09:00  INFO 54361 --- [nio-8080-exec-1] c.e.s.filter.FilterTestController        : /error-test 호출
... // 에러 메시지
```

## registrationBean 통해 filter를 등록 예제

```java
@Configuration(proxyBeanMethods = false)
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<MyFilter> myFilter() {
        FilterRegistrationBean<MyFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new MyFilter());
        registrationBean.addUrlPatterns("/*");
        registrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.FORWARD, DispatcherType.ERROR);
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean<MyOncePerRequestFilter> myOncePerRequestFilter() {
        FilterRegistrationBean<MyOncePerRequestFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new MyOncePerRequestFilter());
        registrationBean.addUrlPatterns("/*");
        registrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.FORWARD, DispatcherType.ERROR);
        return registrationBean;
    }
}
```

### registrationBean 테스트 결과

**FORWARD**

```
GET http://localhost:8080/filter/test-forward

2024-07-05T16:42:17.639+09:00  INFO 60426 --- [nio-8080-exec-4] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/test-forward
2024-07-05T16:42:17.640+09:00  INFO 60426 --- [nio-8080-exec-4] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/test-forward
2024-07-05T16:42:17.654+09:00  INFO 60426 --- [nio-8080-exec-4] c.e.s.filter.FilterTestController        : /test-forward 호출
2024-07-05T16:42:17.660+09:00  INFO 60426 --- [nio-8080-exec-4] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/forwarded
2024-07-05T16:42:17.662+09:00  INFO 60426 --- [nio-8080-exec-4] c.e.s.filter.FilterTestController        : /forwarded 호출
```

**REDIRECT**

```
GET http://localhost:8080/filter/test-redirect

2024-07-05T16:43:49.130+09:00  INFO 60426 --- [nio-8080-exec-1] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/test-redirect
2024-07-05T16:43:49.132+09:00  INFO 60426 --- [nio-8080-exec-1] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/test-redirect
2024-07-05T16:43:49.134+09:00  INFO 60426 --- [nio-8080-exec-1] c.e.s.filter.FilterTestController        : /test-redirect 호출
2024-07-05T16:43:49.148+09:00  INFO 60426 --- [nio-8080-exec-3] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/redirected
2024-07-05T16:43:49.149+09:00  INFO 60426 --- [nio-8080-exec-3] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/redirected
2024-07-05T16:43:49.149+09:00  INFO 60426 --- [nio-8080-exec-3] c.e.s.filter.FilterTestController        : /redirected 호출
```

**ERROR**

```
GET http://localhost:8080/filter/test-error

2024-07-05T16:44:19.005+09:00  INFO 60426 --- [nio-8080-exec-6] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /filter/test-error
2024-07-05T16:44:19.007+09:00  INFO 60426 --- [nio-8080-exec-6] c.e.s.filter.MyOncePerRequestFilter      : MyOncePerRequestFilter - 처리중인 URL= /filter/test-error
2024-07-05T16:44:19.014+09:00  INFO 60426 --- [nio-8080-exec-6] c.e.s.filter.FilterTestController        : /error-test 호출
... // 에러 메시지
2024-07-05T16:44:19.022+09:00  INFO 60426 --- [nio-8080-exec-6] c.example.springbootlab.filter.MyFilter  : MyFilter - 처리중인 URL= /error
```

## 총 테스트 결과

| Filter 방식 | @Component Filter | @Component OncePerRequestFilter | registrationBean Filter | registrationBean OncePerRequestFilter |
| :---       | :---:             | :---:                           | :---:                   | :---:                                 |
| FORWARD    | 1번                | 1번                             | 2번                      | 1번                                   |
| REDIRECT   | 2번                | 2번                             | 2번                      | 2번                                   |
| ERROR      | 1번                | 1번                              | 2번                     | 1번                                   |

- @Component를 사용할 때는 둘 다 동일헀지만, registrationBean을 통한 구현시 차이점을 보였습니다.   
- REDIRECT 시에는 모두 2번씩 호출되는 걸 볼 수 있었는데, 어찌보면 당연합니다. FORWARD, ERROR 와 다르게 REDIRECT는 클라이언트까지 응답이 간다음에 다시 재요청하는 것이기 때문에 필터를 거쳐야 합니다.(거치지 않으면 위험할 수 있습니다)

## 결론

- OncePerRequestFilter는 동일한 요청에 대해 필터가 여러 번 수행되는 것을 방지해줍니다.(명시적)
- OncePerRequestFilter는 사용할 땐, Servletrequest -> HttpServletrequest로 불필요한 타입 캐스팅을 하지 않아도 됩니다.
- 이 외에도 `shouldNotFilter()` 메서드를 이용해서 유연한 필터링이 가능합니다.

## 참조

- [What Is OncePerRequestFilter? - baeldung](https://www.baeldung.com/spring-onceperrequestfilter)

- [Class OncePerRequestFilter - spring-docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html)

- [Spring Filters Series 4 - OncePerRequestFilter in Spring - Youtube](https://www.youtube.com/watch?v=XE6cZnAnrW0&list=PLrHjhj3I5M_ljv8s-Bejj4tbYPjOnjwQp&index=4)

## 전체 테스트 코드

[전체 테스트 코드 깃허브 링크](https://github.com/biddan606/springboot-lab/tree/main/src/main/java/com/example/springbootlab/filter)