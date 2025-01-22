# Vertical Slice Architecture

아키텍쳐가 왜 필요한지에 대해 이야기하고 `Vertical Slice Architecture`가 언제 유용한지에 대해 이야기해보겠습니다.

## 아키텍쳐가 필요한 이유

작은 앱에서는 필요가 없을 수도 있습니다. (코드를 모두 이해하기 간단한 로직이라면)   
하지만 규모가 커질수록 많은 부가기능(`Authorization`, `Validation`, `Data Access`, `Logging`)과 복잡한 비즈니스 로직이 생겨납니다.   
이러한 것들이 규칙없이 생겨나게 되면 코드를 변경하기 두려워집니다. 또한 코드를 이해하기도 어렵습니다. 

## 의존 방향

![alt text](<image/Vertical Slice Architecture/클린 아키텍쳐.png>)

여러 아키텍쳐들이 있겠지만, 가장 널리 알려져있는 클린 아키텍쳐입니다.   
클린 아키텍쳐는 화살표 방향으로 의존하고 있습니다. 
외부는 내부를 의존하고, 가장 안쪽 계층인 `Entites`는 아무것도 의존하고 있지 않습니다.   
이러한 아키텍쳐의 핵심은 이 많은 것들을 구현하라는 것이 아니라 의존 방향을 안쪽으로만 제어한다는 것입니다.

## 내부 참조

![alt text](<image/Vertical Slice Architecture/레이어드 아키텍쳐.png>)

반드시 다음 레이어만을 참조할 필요는 없습니다. 내부의 내부를 다이렉트로 참조해도 됩니다. 반드시 지켜야 할 사항은 내부로만 가야한다는 것입니다.   
이렇게 컨트롤 되어야 순환 참조가 생기지 않고, 코드의 흐름을 알 수 있습니다.

## 이러한 아키텍쳐가 갖는 이점

`클린 아키텍쳐`, `레이어드 아키텍쳐`는 내부로 갈수록 안정적입니다.(`Domain`이 가장 안정적)  
여기서 안정적이라는 말은 다른 어떠한 객체에 변화가 생겨도 상관없다는 것입니다. `Domain`은 아무것도 의존하고 있지 않기 때문에 어떠한 것이 변화해도 영향을 받지 않습니다.

## 안정성(Stability)가 중요한 이유

안정성으로 인해 규칙이 생깁니다. 자신보다 높은 계층의 변화는 자신과 자신 아래의 계층에 영향을 미치지 않습니다. 높은 계층을 참조하고 있는 그 상위의 계층만 검사하면 됩니다.   
만약 `infrastucture` 계층에 변화가 생긴다면, 해당 `infrastucture`를 의존하고 있는 `WebUI`만 검사하면 됩니다.   
복잡한 코드들에서 검사해야 할 범위가 생긴겁니다.   

## 의존 방향 규칙을 만드는 유일한 방법일까?

그렇지 않습니다. 이 또한 단점이 있고, 다른 방법도 존재합니다. 가장 큰 단점은

![alt text](<image/Vertical Slice Architecture/획일화된 구조.png>)

프로젝트의 패키지 구조만 보고 해당 프로젝트가 무엇을 하는지 알 수 없습니다.   
이 말은 프로젝트의 기능을 알기 위해서는 모든 패키지를 뒤져가며, 기능을 분석해야 합니다.   

### 두번쨰 문제점

`기술적 관심사(technical concerns)`로 분리된 `레이어드 아키텍처(Layered architectures)`나 클린 아키텍처(Clean architectures)의 두번째 문제는 응집도에 있습니다.   
두 아키텍처는 단방향으로 낮은 결합도를 지향하지만, 반대로 낮은 응집력을 가집니다.(낮은 결합도엔 낮은 응집력, 높은 응집력엔 높은 결합력과 같은 음양처럼 같이 붙어다님)   
`web` 안에는 모든 `endpoint`들이 존재할테고, 1개의 `endpoint` 요구사항만 변경되더라도 `web` 패키지는 변합니다.   

![alt text](<image/Vertical Slice Architecture/음양.png>)

결합도와 응집력은 음과 양처럼 같이 올라가거나, 같이 내려가기 때문에 낮은 결합도를 얻었다면 낮은 응집도 또한 얻을 수 있습니다.   

## 응집력 높이기

그렇다면 어떻게 응집력을 높이고, 낮은 결합도를 유지할 수 있을까요?   
응집력을 높이는 방법은 2가지가 있습니다.
- 데이터 그룹화, 관련있는 데이터끼리의 묶음(정보적 응집성)
- 관련있는 행동 또는 작업끼리의 묶음(기능적 응집성)

이 중 기능적 응집성을 기준으로 묶어보겠습니다.(기능별)   
이후 왜 데이터가 아닌 기능별로 묶었는지에 대해 이야기해보겠습니다.

![alt text](<image/Vertical Slice Architecture/데이터 묶음 패키지.png>)

위와 같은 코드가 `Commands` `EventHandlers`, `Queies` 와 `기술적`으로 묶여있는 것을

![alt text](<image/Vertical Slice Architecture/기능적 묶기.png>)

각 기능별로 묶어둘 수 있습니다. 더 나아가 `Application`, `WebUI` 등도 기능별로 묶게 된다면, 싱글 파일로는 다음과 같이 나옵니다.

![alt text](<image/Vertical Slice Architecture/싱글파일.png>)

이러한 구조를 띄게 된다면, `CreateTodo` 에 관련된 변경사항이 있더라도 이 파일만을 변경하면 됩니다.(싱글 파일이 싫다면, 같은 폴더에 각 클래스별로 분리하면 됩니다)

### 기능별로 묶은 이유

![alt text](<image/Vertical Slice Architecture/erd.png>)

위 그림은 데이터들을 나열한 ERD입니다.   
그림만 보고는 각 데이터들이 어떤 상호작용을 하는지 알 수 없습니다. 이 데이터들이 어떻게 묶이면 좋을지 알 수 없는 이유는 우리 시스템에서 해당 데이터가 어떤 개념으로 쓰일지 알 수 없기 떄문입니다.

![alt text](<image/Vertical Slice Architecture/같은 이름의 다른 개념들.png>)

다양한 도메인별로 `Product`가 다른 개념으로 쓰일 수 있습니다. `Catalog`에서는 단순히 CRUD이지만, `Sales`에서는 할인 정책, 재고 확인 등 다양한 기능을 지니게 됩니다.   
그래서 우리가 해야할 건 데이터를 보는 것이 아니라 기능과 개념을 봐야 합니다. 

## 바뀐 패키지 구조

![alt text](<image/Vertical Slice Architecture/바뀐 경계1.png>)

`레이어드 아키텍쳐` -> `기능별 아키텍쳐`로 바뀌었으니 이젠 점선과 같은 경계로 변경됩니다.   

![alt text](<image/Vertical Slice Architecture/바뀐 경계2.png>)

그럼 그 안에서 다음과 같이 분리됩니다.

## Vertical Slice Architecture의 오해

지금까지 그림만 보자면 `Vertical Slice Architecture` 는 아무것도 공유하지 않는다고 생각할 수 있습니다. 하지만 그렇지 않습니다.

![alt text](<image/Vertical Slice Architecture/도메인 공유.png>)

도메인이 필요하다면, 여러 `feature`들이 기본 도메인을 공유할 수 있습니다.

## vertical slice는 모든 것을 수직 분할하나?

도메인과 데이터베이스를 보면 해당 영역은 수평으로 되어 있습니다.   
또한, 모든 것을 수직으로 분할하기는 쉽지 않고, 하더라도 오히려 이해하기 힘들 수 있습니다.   
예를 들어, `springboot`에서 `config`, `interceptor` 등의 역할을 수직 분할하게 되면, 어떤 `interceptor`가 있고, 어떤 설정이 되어있는지 알기 힘듭니다. 이 경우에는 수평 분할이 적합합니다.

`vertical slice`의 핵심은 개별 요청에 따라 응집한다는 것입니다. 
`vertical slice`보다 `feature architecture`가 더 어울릴 수도 있습니다.

## 결론

- 아키텍쳐에서 중요한 것은 의존 방향이다.
- 레이어드 아키텍쳐는 어떤 프로젝트에서도 사용할 수 있는 일관성이 있지만, 실용적이지 않을 수 있다.
- 결합력과 함께 응집도도 생각해야 한다.
- `Vertical Slice Architecture`를 사용하면 응집도를 높일 수 있다.
- 데이터가 아닌 기능과 개념에 초점을 맞춰야 한다.

## 참조

[Vertical Slice Architecture, not Layers! - CodeOpinion Youtube](https://www.youtube.com/watch?v=L2Wnq0ChAIA)

[Vertical Slice Architecture - Milan Jovanović](https://www.milanjovanovic.tech/blog/vertical-slice-architecture)