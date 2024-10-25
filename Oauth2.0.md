# Oauth2.0

타사의 애플리케이션이 사용자의 보호된 리소스를 액세스할 수 있도록 설계되었습니다.   
정확히는 비밀번호 없이 타사의 애플리케이션이 사용자의 보호된 리소스를 접근할 수 있도록 설계되었습니다.   

먼저 비밀번호 있는 버전을 살펴보고, Oauth2.0 버전에 대해 알아보겠습니다.

## 비밀번호를 통한 접근 버전

![alt text](<image/Oauth2.0/패스워드를 통한 리소스 접근.png>)

1. `User` -> `3rd Party App`에 접속하여 `Server`의 아이디와 비밀번호를 입력합니다.
2. `3rd Party App` 가 넘겨받은 아이디와 비밀번호를 이용해 `Authroziation Server` 에 인증합니다.
3. 인증이 완료되었으니, `Authroziation Server` -> `3rd Party App` 에게 인증 토큰을 넘겨줍니다.
4. `3rd Party App` 는 인증 토큰을 통해 `Autorization Server` 에 `Resourse` 를 요청합니다.
5. `Autorization Server` -> `Resourse Server` 에서 `Resourse` 를 가져와 `3rd Party App` 에게 넘겨줍니다.

### 문제점

2가지 문제점이 있습니다.

- `User`의 아이디와 비밀번호를 제 3자인 `3rd Party App` 이 알게 됩니다. 악용할 우려가 있습니다.
- `3rd Party App` 이 `User` 가 원하는 정보만 가져올지 알 수 없습니다. 그 이상의 정보를 가져와 저장할 수 있습니다.

비밀번호를 통한 접근은 이러한 문제점들이 있습니다.   
`Oauth2.0` 을 사용하면 이러한 문제점을 해결할 수 있습니다.

## Oauth2.0을 통한 방법

![alt text](<image/Oauth2.0/oauth를 통한 접근.png>)

1-1. `User` -> `Resource` 에 접근하려고 `3rd Party App` 에 요청합니다.   
1-2. `3rd Party App` 는 `resource` 접근을 위해 `authorization(권한)` 이 필요합니다.   
1-3. `3rd Party App`는 `User`에게 `Authorization Server` 로그인 링크를 줍니다.   

2-1. `User` -> `Authorization Server` 에 로그인합니다.   
2-2. `Authorization Server`는 로그인 값들을 검증합니다.   
2-3. 검증이 올바르므로, `Authorization Server` -> `User`에게 어떤 정보들의 권한을 부여할 것인지를 묻습니다.   

3-1. `User`는 `Resource` 권한을 체크합니다.   
3-2. `Authorization Server`는 정보 권한을 획득할 수 있는 인증 코드를 발급합니다.

4-1. `User` -> `3rd Party App`에게 인증 코드를 건네줍니다.

5-1. `3rd Party App` -> `Authorization Server`에게 인증 코드를 주고, 액세스 코드를 받습니다.

6-1. `3rd Party App` -> `Resource Server`에 액세스 토큰과 함께 정보를 요청합니다.   
6-2. `Resource Server`는 액세스 토큰이 올바르므로 정보를 넘겨줍니다.

7-1. `3rd Party App` -> `User`에게 정보를 다시 전달합니다.

이전 방법보다 번거롭지만, 서로의 정보를 안전하게 지키면서 타사 서비스의 자원을 이용할 수 있습니다.

## 참조

- [OAuth 2.0 explained with examples - ByteMonk](https://www.youtube.com/watch?v=ZDuRmhLSLOY)

- [OAuth 2 Explained In Simple Terms - ByteByteGo](https://www.youtube.com/watch?v=ZV5yTm4pT8g)
