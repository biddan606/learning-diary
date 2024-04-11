# Servlet

위키피디아에서는 자바를 사용하여 웹페이지를 동적으로 생성하는 서버측 프로그램 혹은 그 사양이라고 표현되어 있습니다.   
하지만 저는 이 표현이 애매하다고 생각하여 다시 재정의해보겠습니다.   
자바에서 서블릿은 `HttpServlet` 클래스를 상속 받아 `DefaultServlet`, `DispatcherServlet`, `HttpJspBase` 등 재정의하여 사용합니다.
또한, `DefaultServlet` 과 같이 정적인 리소스를 처리하는 서블릿도 존재합니다.   
그렇기에 제가 생각하는 서블릿의 정의는 HTTP 요청을 처리하는 클래스가 조금 더 명확한 것 같습니다.

## 서블릿이 어떻게 HTTP 요청을 처리하는가?

서블릿은 자체만으로는 HTTP 요청을 받아 처리하지 못합니다.   
서블릿은 단순히 `Request` 와 `Response` 인자가 들어올 수 있고, `GET`, `POST`, `DELETE`, `PUT` 요청에 따라 처리하여 `Request` 값들을 처리하여 `Response` 에 담아두겠습니다. 라는 약속 뿐입니다.   
그래서 HTTP 요청을 받고, 서블릿에 `Request` 와 `Response` 인자를 넣어준 뒤, 응답을 처리하는 객체가 필요합니다. 이것을 서블릿 컨테이너가 처리합니다.

## 서블릿 컨테이너(= 웹 컨테이너)

HTTP 요청을 받아 서블릿으로 보내고, 응답을 받아 클라이언트에서 반환하는 역할을 합니다.   
쉽게 말해, JVM 위에 돌아가는 애플리케이션 서버라고 할 수 있습니다.

## 서블릿 컨테이너 동작 방식

1. 서블릿 컨테이너가 클라이언트의 요청을 받습니다.
2. 요청을 `Request` 객체로 변환하고, 응답을 저장할 `Response`를 만듭니다.
3. 요청을 처리할 스레드를 할당하거나 생성합니다.
4. 스레드에 `Request`, `Response` 를 담아 path에 맞는 서블릿을 호출합니다.
5. Response 응답을 클라이언트에게 반환합니다.

[서블릿 컨테이너 동작 방식에 대한 자세한 그림 설명](https://finerss.tistory.com/11)

## 스레드를 만드는 이유

서블릿 자체는 싱글톤입니다. 서블릿 필드를 통해 상태를 저장하면 서로 다른 요청이 서로의 상태를 변경할 수 있습니다.   
상태를 스레드의 스택에 저장하여 관리해야 되기 떄문에 스레드를 생성합니다.

## 서블릿의 생명 주기

path에 해당하는 서블릿 클래스 정보를 web.xml에 저장하고, web.xml 정보를 서블릿 컨테이너가 사용합니다.
그래서 서블릿의 생명주기도 서블릿 컨테이너가 관리합니다.

### 생성

1. 서블릿 컨테이너가 서블릿을 처음 호출할 때, 서블릿을 인스턴스화하고 init() 메서드를 호출합니다.
2. 서블릿을 다음 번에 호출할 때는 이전에 만들어진 인스턴스를 호출합니다.(싱글톤)

### 실행

- 만들어진 서블릿 인스턴스에 service() 메서드를 호출합니다.

### 제거

- 서블릿 컨테이너가 종료될 때, 서블릿의 destroy() 메서드를 호출합니다.   


위에서 서블릿 컨테이너를  "JVM 위에 돌아가는 애플리케이션 서버" 라고만 정의했는데, 추가로 정의할 수 있을 것 같습니다.   
서블릿 컨테이너는 JVM 위에서 실행되는 애플리케이션 서버로, 요청 스레드 관리, path와 서블릿 매핑, 요청 응답 생성 등 서블릿 실행에 필요한 환경을 제공합니다.   
또한, 서블릿의 생명주기 관리도 합니다.

## Spring boot에서는 톰캣 설정을 안해주는데 왜 잘 동작하는가?

### 스프링 부트

1. 스프링 부트는 실행 시, `AutoConfiguration` 을 통해 내장 톰캣을 자동적으로 구성합니다. 
2. 내장 톰캣에 모든 path("/")를 `DispathcerServlet` 으로 등록합니다.
3. @Controller 어노테이션이 붙은 빈들을 `DispathcerServlet` 과 매핑시킵니다.

스프링 부트에는 일반적으로 정적 컨텐츠를 관리하는 `DefaultServlet` 과 컨트롤러와 매핑되어 있는 `DispathcerServlet` 이 있습니다.   

### 스프링(전통적인 WAR 배포)

스프링은 내장 톰캣이 존재하지 않습니다. 수동으로 서블릿 컨테이너와 연결시켜주어야 합니다.

1. 스프링 프로젝트를 `war` 파일로 패키징합니다.
2. 서블릿 컨테이너 디렉토리에 `war` 파일을 복사하여 배포합니다.
    - 톰캣의 경우, webapps 디렉토리에 복사합니다.

## JSP는 어떻게 통신할까?

jsp는 특별한 형태의 서블릿입니다. 아래에 설명은 jsp 페이지를 처음 요청했을 경우입니다. 재요청할 경우 일반 서블릿과 동일하게 동작합니다.

1. 클라이언트로부터 HTTP 요청이 서블릿 컨테이너에 도착합니다. 해당 요청이 JSP 페이지인 경우, JSP 파일 처리를 시작합니다.
2. jsp 파일을 이를 Java 소스 코드로 변환합니다.   
    - Java 소스 코드는 `HttpServlet`를 상속받은 `HttpJspBase` 클래스의 형태를 취하며, JSP 페이지 내용을 구현합니다.   
3. Java 소스 코드 class 파일로 컴파일합니다. 이 때 문법적 정확성 검사도 수행합니다.
4. class 파일이 서블릿 컨테이너에 의해 로드되고, 인스턴스화된 후 실행됩니다.

- jsp 파일이 인스턴스화 되는 지점은 war 파일로 패키징 될 때가 아니라, 서블릿 컨테이너가 해당 요청을 받을 때 처리됩니다.(최적화가 필요한 경우, jsp 파일을 사전 처리(pre-compilation) 할 수도 있습니다)   
- 스프링 부트에서 기본적으로 제공하지 않습니다. 사용을 원할 경우 의존성을 주입하여 사용해야 합니다.

## 참조

[Servlet Container와 Servlet의 관계 - 박재성](https://www.youtube.com/watch?v=aP4Lw3SfffQ&list=PLqaSEyuwXkSoeqnsxz0gYWZMihw519Kfr&index=6)   

[Servlet & Servlet Container - 강현구](https://www.youtube.com/watch?v=Xx9BXrzNHn8)   

[1. Servlet이 실행되는 환경 이해하기 - 부부개발단](https://www.youtube.com/watch?v=yXFfEqm-JPg)   
[2. Http 프로토콜과 서블릿이 실행되는 과정 이해하기 - 부부개발단](https://www.youtube.com/watch?v=20U7tScxB1I)   
[3. Servlet, JSP라이프 싸이클, Servlet vs JSP , 포워딩 맛보기 - 부부개발단](https://www.youtube.com/watch?v=v7OF35PjwVg)   

[[Spring] Servlet, Servlet Container, Spring MVC 정리 - 제이온](https://steady-coding.tistory.com/599#google_vignette)   

