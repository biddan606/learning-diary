# IoC(Inversion Of Controll)란?

말 그대로 "제어의 역전"이라는 의미입니다. 프레임워크에서 사용되고, 제어 흐름이 사용자가 아닌 프레임워크에 있는 것을 말합니다.
한데, 프레임워크에서 "제어의 역전"은 어떻게 보면 당연합니다. 프레임워크는 기본 틀을 제공해주고 제공해준 틀을 이용하기 때문에, 프레임워크를 이용하면 통제권은 프레임워크에 있을 수 밖에 없을 것 같습니다.   
중요한 점은 어떤 통제권을 프레임워크가 가지고 있는가 입니다. 보통 IoC라고 말하면 '플러그인을 주입할 수 있는 권한' -> 객체 관리(객체 생성, 초기화, 소멸 등)을 말합니다. 간단하게 말해서, '객체의 생명주기 관리를 프레임워크가 담당하는 패턴' 이라고 할 수 있습니다.

## 스프링에서는 왜 IoC를 필요한가?

스프링은 경량 컨테이너라서 상황에 따라 플러그인을 교체할 수 있어야 합니다.

![alt text](<image/스프링에서의 IoC/MovieLister와 MovieFinder 의존관계.png>)

```java
public class MovieLister {
  private MovieFinder finder;
  
  public MovieLister() {
    this.finder = new ColonDelimitedMovieFinder("movies1.txt");
  }
}
```

위 코드와 같이 `MovieLister` 가 텍스트 파일에서 영화를 검색하는 `MovieFinderImpl` 을 의존하고 있지만, 데이터베이스에서 영화를 검색하는 플러그인으로도 교체될 수 있습니다.   
직접 코드를 수정해서 교체할 수 있지만, 그럴 경우 변경이 매우 잦게 됩니다.
이를 해결하기 위해 IoC가 필요합니다.(프레임워크가 객체 관리를 담당하고 조건, 상황에 따라 주입이 필요함)

**경량 컨테이너**: 낮은 리소스와 빠른 시작, 간편한 설정을 통해 실행할 수 있는 컨테이너를 말합니다. 

## IoC의 종류

IoC의 패턴으로써 `Dependency Injection` 과 `Service Locator` 패턴이 있습니다.   
스프링에서 자주 사용하는 방식은 `Dependency Injection` 이므로 DI부터 알아보고 `Service Locator` 에 대해 알아보겠습니다.

# DI(Dependency Injection)

![alt text](<image/스프링에서의 IoC/MovieLister와 MovieFinder와 Assembler.png>)

DI는 인터페이스를 통한 구현을 하고 이를 어셈블러를 통해 주입받는 형태입니다. 그리고 주입받는 형태는 생성자 주입, setter 주입, interface 주입이 있습니다.

**생성자 주입**   
```java
@Component
public class MovieLister {
  private final MovieFinder finder;
  
  public MovieLister(MovieFinder finder) {
    this.finder = finder;
  }
}
```

**setter 주입**   
```java
@Component
public class MovieLister {
  private MovieFinder finder;
  
  @Autowired
  public setFinder(MovieFinder finder) {
    this.finder = finder;
  }
}
```

**생성자 주입**   
```java
@Component
public class MovieLister {
  @Autowired
  private MovieFinder finder;
}
```

## 어떤 주입 방식을 택할까?

먼저 인터페이스 주입에 대해 이야기해보겠습니다. 가장 심플하고 코드도 짧습니다. 하지만 그만큼 리플렉션과 같은 복잡한 방법을 이용해서 의존성을 주입해줍니다.   
인터페이스 방식은 코드 그 이상의 동작을 수행해야 하고, 그에 따라 테스트 코드에서 객체를 목킹하기 어렵습니다.(테스트 클래스는 사용해도 무방)   

setter 주입을 사용하기 위해서는 불변하지 않아야 하는데, 불변하지 않다는 것은 의미가 모호합니다. 추후 이 코드를 보는 개발자는 특별한 이유가 있다고 생각할 것이고, 그래야 합니다. 또한, setter를 열어둠으로써 캡슐화도 깨지게 됩니다. 특별한 이유가 있지 않고서는 사용할 이유가 없습니다.

생성자 주입은 final이고, 생성자를 통해 주입 받으므로 코드에 전부 들어납니다. 가장 선호하고 권장되는 방식입니다. 하지만 이러한 특성 때문에 생성자의 매개변수가 많아질 수 있습니다. 첫 시작은 생성자 주입을 통해 하고 상황에 따라 다른 주입 방식을 택하는 것이 올바릅니다.

# Service Locator

![alt text](<image/스프링에서의 IoC/서비스 로케이터 방식.png>)

전체 객체 관리를 서비스 로케이터에게 맡기고, 서비스 로케이터에게 필요한 객체를 요청하는 방식입니다.   
서비스 로케이터 방식도 DI와 마찬가지로 구체적인 구현체의 종속을 제거합니다.

```java
@Component
public class MovieLister {
  private final MovieFinder finder;
  
  public MovieLister(ServiceLocator locator) {
    this.finder = locator.getService(MovieFinder.class);
  }
}
```

모든 객체가 `ServiceLocator` 에 종속적이고, 생성자의 매개변수가 `ServiceLocator` 라서 필요한 의존성에 대해 덜 명시적이라는 단점이 있습니다.   

**약간의 의견**   
스프링에서 IoC로 DI 방식을 채택한 이유는 위에서 언급한 `ServiceLocator` 약간의 단점 때문이라고 생각합니다. DI가 `ServiceLocator` 의 업그레이드 버전이라 느껴집니다.   
그만큼 구현의 어려움은 있을 것 같습니다. 만약 제가 프레임워크를 만든다면, 초반에는 `ServiceLocator` 방식으로 구현하는 것이 좀 더 쉽지 않을까 생각합니다.

# 참조

- [Inversion of Control Containers and the Dependency Injection pattern - martinfowler](https://martinfowler.com/articles/injection.html)

- [Dependency Injection Spring - Dan Vega Youtube](https://www.youtube.com/watch?v=TBlB2_4_Sqo)
- [Spring Constructor Injection - Dan Vega Youtube](https://www.youtube.com/watch?v=aX-bgylmprA)

- [Intro to Inversion of Control - baeldung](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)