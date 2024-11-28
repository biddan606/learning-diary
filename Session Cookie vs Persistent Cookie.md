## Session Cookie

브라우저가 닫히기 전까지만 유지되는 쿠키

## Persistent Cookie 

설정한 기간동안 유지되는 쿠키

## 설정하는 방법

`Max-Age`, `Expires`를 통해 설정할 수 있습니다.   
`Max-Age`: 유지할 시간을 초를 기준으로 입력함   
`Expires`: 만료되는 시간을 입력(반드시 GMT 기준)   

### `Max-Age`를 기준으로 각 세션 설정 방법

`Session Cookie`: `Max-Age`를 음수로 표기합니다.(-1)   
`Persistent Cookie`: 양수로 유지할 시간을 설정
`0일 경우`: 쿠키를 저장하지 않고 즉시 삭제

## `Max-Age` vs `Expires`

두 `Attributes`는 같은 역할을 한다는 것을 알 수 있습니다. 그럼 어떤 것을 선호할까요?   
일단 구글은 `Max-Age`, `Expires` 2개의 속성이 모두 되어있다면 `Max-Age`를 우선적용합니다.   
IE의 구버전에서는 `Max-Age`를 인식하지 못하므로 `Expires`가 필수입니다.   
여러 브라우저마다 작동 방식이 다르므로 2개 다 입력해주는 것이 좋습니다. 하지만 1가지만 입력해야 한다면 `Max-Age` 방식을 선호합니다.(개인적)   
- `Expires`는 0초를 입력하기 쉽지 않습니다. 서버에서 `Expires` 시간을 설정해주어 브라우저로 넘겨주지만,   
브라우저가 받는 시간과 서버가 넘겨주는 시간이 일치하지 않을 수 있습니다.(`Expires`가 절대시간이기 때문에 발생하는 문제)
- `Max-Age`가 직관적입니다. 초를 기준으로 넘겨주기 때문에 직관적으로 얼마나 유지하는지 알 수 있습니다. 반대로 `Expires`는 `GMT`를 기준으로 하기 때문에, 직관적이지 않습니다.(변환을 해주어야 함)

### `Expires`가 GMT 기준인 이유

브라우저, 서버 등 각자가 어느 시간대를 기준으로 두는지 알 수 없습니다.   
`Expires`는 클라이언트 시간대 기준으로 동작하기 때문에 특정 시간대 기준이 없다면, 브라우저와 서버가 다르게 만료될 수 있습니다.   
`GMT`를 기준으로 둠으로써 동일한 만료시간을 제공합니다.   

## 예외

- 크롬의 경우 최대 만료시간은 400일입니다. 더 긴 만료시간을 두더라도 브라우저에서 만료해버립니다.(보안적 이유인 듯?)
- `Max-Age`, `Expires`를 모두 설정하지 않을 경우 `세션 쿠키`가 생성됩니다.

## 참조

- [Session Cookies vs Persistent Cookies: Understanding the Differences [Updated February 2024]](https://secureprivacy.ai/blog/session-cookies-vs-persistent-cookies)

- [Expires and Max-Age Cookie Attributes | Session Cookies and Persistent Cookies - Pedagogy Youtube](https://www.youtube.com/watch?v=cRpQd7Fxpo8)

- [Timezone for Expires and Last-Modified HTTP Headers](https://stackoverflow.com/questions/1638932/timezone-for-expires-and-last-modified-http-headers)
