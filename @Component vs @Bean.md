# 스프링에서 객체 사용하는 방법

스프링을 사용하는 개발자는 객체 관리를 직접하지 않습니다.   
`ApplicationContext` 를 이용하여 객체를 관리하고 DI를 통해 주입받습니다.(스프링에서는 이를 Bean이라고 부릅니다)   
그래서 개발자는 클래스를 구현한 뒤 `ApplicationContext` 가 관리할 수 있도록 등록해야 합니다.   
등록하는 방법은 @Component를 이용하는 방법과 @Bean을 이용하는 방법 2가지가 있습니다. 어떤 상황에서 이용하면 좋을지 알아보겠습니다.

## @Component

`@Component` 는 클래스 레벨에서 선언할 수 있는 어노테이션입니다.   
클래스 레벨에 선언하는 어노테이션이기 때문에 직접 만든 클래스에만 선언할 수 있습니다.   
하지만 `@Component` 선언만으로는 아무런 역할도 하지 않습니다. `@Component` 이 선언된 클래스가 빈으로 등록되기 위해서는 컴포넌트스캔이 되어야 합니다.(`@ComponentScan` 을 통해 컴포넌트스캔을 할 수 있습니다)   
기본적으로 `@SpringBootApplication` 내부에 `@ComponentScan` 이 선언되어 있어, Application 클래스와 같은 패키지라면 빈으로 등록할 수 있습니다.

### 구체적인 역할 명시

미리 `@Component` 결합하여 만들어진 메타 어노테이션 `@Controller`, `@Serivce`, `@Repository` 를 통해 구체적인 역할을 명시할 수도 있습니다.   

- **@Controller**: 웹 요청을 처리하는 프레젠테이션 계층의 컴포넌트를 나타냅니다. 요청 핸들러로 인식합니다. 요청 핸들러로 인식하여 `@ResponseBody` 와 같은 편의 기능을 사용할 수 있습니다.

```java
@Component
@ResponseBody // @Component에서는 @ResponseBody가 동작하지 않습니다.
@RequestMapping("/hello")
public class HelloController {

    @GetMapping
    public String get() {
        return "success";
    }
}
```

- **@Serivce**:  `@Component` 와 기능적인 차이는 없지만, 향후 스프링에서 추가 기능을 제공할 수도 있습니다.

- **@Repository**: 데이터 액세스 예외를 Spring의 DataAccessException으로 자동 변환합니다.

## Bean

`@Configuration` 이 선언된 클래스 내부에 메서드 형태로 빈을 등록하는 방식입니다.

```java
@Configuration
public class WebConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplateBuilder().build();
    }

}
```

외부 라이브러리, 미리 정의된 클래스를 빈으로 등록해야 할 경우 사용할 수 있습니다.(`@Component` 를 사용하기 위해서는 코드가 수정되어야 하는데, 수정이 불가능하므로)   

또한, 람다를 사용하고 싶을 경우에도 가능합니다.   
```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner commandLineRunner() {
        return args -> {
            System.out.println("Hello 👋🏻");
        };
    }

}
```

## 결론

**@Component**:
- 클래스 내부에 선언하므로, Config 파일이 필요없고 클래스 내부 코드만으로 빈으로 관리되길 원하는지 알 수 있습니다.
- @Component이 Bean으로 등록되기 위해서는 @ComponentScan이 필요합니다.
- `@Controller`, `@Serivce`, `@Repository` 를 통해 구체적인 역할을 명시할 수도 있습니다.

**@Bean**:
- 코드를 수정할 수 없을 경우, `@Bean`을 이용해 등록할 수 있습니다.
- `@Configuration` 클래스 내부에 메서드의 형태로 등록하기 때문에, 함께 관리되어야 하는 Bean들을 그룹화하여 관리할 수 있습니다.
- 람다를 통해 간단하게 등록할 수도 있습니다.

## 참조

- [Spring @Component Annotation - baeldung](https://www.baeldung.com/spring-component-annotation)
- [@Component vs @Repository and @Service in Spring - baeldung](https://www.baeldung.com/spring-component-repository-service)

- [@Component vs @Bean! - Dan Vega Youtube](https://www.youtube.com/watch?v=CWEQ-1vff1o)