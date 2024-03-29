# HTTP 응답코드에 대해 설명해 주세요.

HTTP 응답 코드는 클라이언트의 요청을 처리한 결과를 알려줍니다. 잉답은 5개의 그룹으로 나누어집니다.

[HTTP 상태 코드](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)

## 범위별 결과

**임시 응답(100~199)**   
**성공 응답(200~299)**: 요청이 성공적으로 수행되었음을 알려줍니다.   
**리디렉션 메시지(300~399)**: 요청한 리소스가 다른 위치로 이동되었음을 나타냅니다.   
**클라이언트 에러 응답(400~499)**: 클라이언트의 요청이 잘못되었음을 나타냅니다.   
**서버 에러 응답(500~599)**: 서버가 유효한 요청을 처리할 수 없음을 나타냅니다.

## 주로 사용하는 응답 코드

### 200번대
**200 OK**: 요청이 성공적으로 처리되었음을 나타냅니다.   
**201 Created**: 요청이 성공적으로 이행되었고, 새로운 리소스가 생성되었음을 나타냅니다.   
**204 No Content**: 요청이 성공적으로 처리되었으나 클라이언트에 전달할 콘텐츠가 없음을 나타냅니다.   

### 300번대
**301 Moved Permancently**: 요청된 리소스가 영구적으로 새 위치로 이동했음을 나타냅니다.   
**302 Found**: 요청된 리소스가 일시적으로 다른 위치로 이동했음을 나타냅니다.   

### 400번대
**400 Bad Request**: 서버가 요청을 이해할 수 없음을 나타냅니다.(잘못된 형식일 때 발생)   
**401 Unauthorized**: 요청이 인증을 필요로 함을 나타냅니다. 일반적으로 로그인이 필요한 리소스에 접근하려 할 때 사용됩니다. `WWW-Authenticate` 헤더를 통해 클라이언트에서 필요한 인증 방식을 알려줘야 합니다.(필수)
**403 Forbbiden**: 서버가 요청을 이해했지만 거부함을 나타냅니다. 인증과는 무관하게 리소스에 대한 접근 권한이 없음을 의미합니다.   
**404 Not Found**: 서버가 요청한 리소스를 찾을 수 없음을 나타냅니다.   
**429 Too Many Requests**: 클라이언트가 지정된 시간 동안 허용된 요청 수를 초과했음을 나타냅니다.   
`Retry-After`(다음 요청 가능 시간), `X-RateLimit-Limit`(지정된 시간 동안 수행할 수 있는 최대 요청의 수), `X-RateLimit-Remaining`(아직 수행할 수 있는 요청의 수), `X-RateLimit-Reset`(요청 한도가 리셋되는 시간, 보통 에포크 시간으로 표시) 등의 헤더를 줄 수 있습니다.

#### 401 Unauthorized 와 403 Forbbiden 차이
`401 Unauthorized` 은 요청한 리소스에 대한 인증이 필요하다는 것이고, `403 Forbbiden` 는 요청한 리소스에 대한 접근 권한이 없음을 나타냅니다. 서버가 클라이언트의 인증을 인식하고 있으나, 인증된 사용자에게도 리소스에 대한 접근을 허용하지 않음을 의미합니다.(권한 부족)

### 500번대
**500 Internal Server Error**: 서버가 정의되지 않은 서버 에러로 인해 요청을 처리할 수 없음을 나타냅니다.   
**502 Bad Gateway**: 서버가 게이트웨이 또는 프록시 역할을 하며, 상위 서버로부터 유효하지 않은 응답을 받았음을 나타냅니다.
**503 Service Unavailable**: 서버가 일시적으로 서비스를 제공할 수 없음을 나타냅니다. 일반적으로 서버 과부하나 유지 보수로 인해 발생합니다.


## 필요하다면 저희가 직접 응답코드를 정의해서 사용할 수 있을까요? 예를 들어 285번 처럼요.

기술적으로 가능합니다. 하지만 일반적으로 권장되지 않습니다. 정의되지 않은 코드이므로 클라이언트나 중간 매개체에서 올바르지 않게 해석될 수 있기 때문입니다.

## 그럼에도 불구하고 사용자 정의 응답 코드를 사용하는 이유는 무엇일까?

표준 HTTP 코드만으로 특정 상황을 충분히 설명할 수 없거나 추가적인 정보가 필요할 경우 활용할 수 있습니다. 그러나 이 경우에도 표준 코드를 우선적으로 고려하는 것이 좋습니다.

## 사용자 정의 응답 코드의 예시로 285번을 들었는데, 이러한 코드를 실제 환경에서 구현하기 위해 어떤 절차를 따라야 하나?

285번 코드에 대한 상태를 명확히 정의해야 합니다. API 문서나 개발자 가이드에 명시하여 사용자가 이해할 수 있도록 해야 합니다.
