# Springboot-gracefulshutdown

스프링 부트의 `graceful-shutdown` 기능 동작에 대해 정리한 글입니다.   

## graceful-shutdown이란?

`graceful-shutdown`은 프로세스가 종료할 때, 현재 작업들을 안전하게 완료하고 종료하는 것을 의미합니다.   
일반적으로 종료 전에 받은 요청은 수행하고, 종료 이후에 들어온 요청은 받지 않습니다. +종료 전에 받은 요청은 완료해야 하므로 일정 시간 텀을 둡니다.

## shutdown이 graceful해야 하는 이유

- **트랜잭션 완결성**: 애플리케이션이 종료되기 전에 트랜잭션이 완료됨을 보장할 수 있어야 합니다. 트랜잭션이 완료되지 않고 종료되면, 데이터의 일관성이 보장되지 않을 수 있습니다.

- **리소스 정리**: 종료되기 전 수행해야 되는 로직들이 존재할 수 있습니다. 리소스를 정리하고 확인하는 로직이 있을 수 있고, 리소스 또한 종료까지 시간이 걸린다면 애플리케이션 또한 일정 시간이 걸린 후에 종료되어야 합니다.

- **사용자 요청 완료**: 올바른 요청을 보낸 사용자는 서버 꺼짐과 상관없이 응답을 받음을 보장해주어야 합니다.

## springboot의 graceful shutdown의 동작 방식

- 종료 이전에 들어온 요청을 처리합니다. `timeout` 내에 요청을 응답하지 못하는 경우 빈 응답값을 반환합니다.

- 종료 작업 중 들어온 요청들은 받지 않습니다.(springboot 기본 웹서버인 톰캣 기준, 언더토우는 서비스 이용 불가 코드인 503 반환)

- `spring.lifecycle.timeout-per-shutdown-phase` 시간과 `server.shutdown=graceful`를 설정하였더라도 응답할 요청이 없다면 즉시 종료합니다.

## 주의할 점

- `graceful-shutdown`을 설정하였더라도 모든 요청을 처리하는 것은 아닙니다. 복구 계획을 세워두는 것이 중요합니다.

- `spring.lifecycle.timeout-per-shutdown-phase` 시간을 크게 잡았더라도, 처리할 요청이 없다면 종료할 수 있습니다. 시간을 넉넉히 잡는 것이 좋아보입니다.

- 웹서버에 따라 다르게 동작할 수 있습니다.(제티, 리액터 네티, 톰캣 <=> 언더토우)

- 반드시 `SIGTERM(15)`으로 종료하여야 합니다. `SIGKILL(9)`은 프로세스를 즉시 종료하는 시그널이라 `SIGKILL(9)`로 종료할 경우, `graceful-shutdown`을 설정하더라도 즉시 종료합니다.

## 결론

- `graceful-shutdown`을 사용하고 시간을 넉넉히 잡자

- 종료 시그널은 `SIGTERM(15)`!!!

- `graceful-shutdown`은 만능이 아니다. 예상치 못한 경우가 있을 수도 있으므로, 복구 전략을 세워두어야 한다.

## 참조

- [Spring Boot Graceful shutdown video - Mike Møller Nielsen Youtube](https://www.youtube.com/watch?v=wbM4x3ESeos)
- [Springboot-gracefulshutdown code - Mike Møller Nielsen github](https://github.com/ekim197711/springboot-gracefulshutdown)

- [Springboot docs - Graceful Shutdown](https://docs.spring.io/spring-boot/reference/web/graceful-shutdown.html)

