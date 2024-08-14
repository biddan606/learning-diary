# 커밋 메시지

Conventional Commits 1.0.0의 규칙을 따라가고, 규칙에 대한 요약입니다.

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

기본 틀은 위와 같이 가져갑니다.   
## `<type>[optional scope]`

type은 커밋의 종류를 나타냅니다.   
`feat`, `fix`와 같은 타입을 명시합니다.(해당 커밋에 대해 한 일)   
`fix` : 버그를 패치한 경우   
`feat` : 새로운 기능을 도입한 경우   

BREAKING CHANGE(!): 획기적인 변경사항이 있는 경우에 추가하여 강조할 수 있습니다.   
예를 들어, API 엔드포인트 변경, API 요청 또는 응답 형식 변경, 아키텍쳐 구조 변경, 자바 버전 업그레이드, 데이터베이스 스카마 변경 등   
`<type>[optional scope]` 부분에 BREAKING CHANGE로 시작하는 문장을 추가하거나, `<type>[optional scope]` 뒤에 !가 붙여 표현합니다.   

`feat`, `fix` 이외 타입: `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:` 등 여러 타입이 올 수 있습니다.   

`build` : 빌드 시스템이나 외부 종속성에 영향을 주는 변경사항,   
- build: Gradle 버전을 7.0에서 7.2로 업그레이드   
- build: Spring Boot 버전을 2.5.5에서 2.6.0으로 업데이트   
- build: lombok 의존성 추가   


`chore` : 소스 코드나 테스트를 직접 수정하지 않는 유지보수 작업   
프로젝트 구조 변경:   
- chore: src/main/java 디렉토리 구조 정리   
- chore: 사용하지 않는 리소스 파일 제거   

빌드 스크립트 수정 (기능에 영향을 주지 않는 경우):   
- chore: build.gradle에 주석 추가   

프로젝트 문서 관리:   
- chore: CONTRIBUTING.md 파일 추가   
- chore: GitHub 이슈 템플릿 업데이트   
- chore: README.md 파일 오타 수정   

Git 관련 설정:   
- chore: .gitattributes 파일 추가   
- chore: .gitignore 규칙 최적화   
- chore: .gitignore 파일 업데이트로 build 폴더 제외   

CI/CD 관련 (주요 변경이 아닌 경우):   
- chore: CI 스크립트의 들여쓰기 수정   
- chore: GitHub Actions 워크플로우 파일 이름 변경   

변경사항이 코드의 동작이나 성능에 영향을 미치지 않을 때, `chore`를 사용합니다.   
몇몇 예시들을 보면 `build`, `ci`, `docs`로 구체적인 타입을 사용하거나 `rename`, `git`과 같은 추가 타입을 선언하여 하는 것이 고려해볼 수 있습니다.   


`ci`: CI 구성 파일 및 스크립트 변경
- ci: Jenkins 파이프라인 스크립트 추가
- ci: GitHub Actions 워크플로우에 Gradle 캐싱 적용
- ci: SonarQube 분석 단계 Travis CI에 추가


`docs`: 프로젝트의 문서화와 관련된 변경사항(코드 내에 주석의 경우 docs, chore 선택 개인적으로 문서가 아니므로 chore를 선호) 
- docs: README에 프로젝트 설정 가이드 추가
- docs: application.properties 설정 옵션 문서화
- docs: API 엔드포인트 사용법 문서화
- docs: 사용자 매뉴얼 v2.0 업데이트


`style` : 코드의 의미에 영향을 주지 않는 스타일 변경(코드 포맷팅, import 제거, 파일 끝 개행 추가 등)
- style: UserController 클래스의 들여쓰기 수정
- style: 일관된 중괄호 스타일로 ProductService 리팩토링
- style: 사용하지 않는 import 문 제거


`refactor` : 코드의 내부 구조를 개선하지만 외부 동작은 변경하지 않는 경우
- refactor: OrderService의 주문 처리 로직을 별도 메서드로 추출
- refactor: UserAuthentication에 전략 패턴 적용하여 인증 방식 유연성 개선
- refactor: ProductController와 OrderController의 중복 검증 로직을 공통 유틸리티 클래스로 이동
- refactor: PaymentProcessor 클래스 계층 구조를 인터페이스 기반으로 변경
- refactor: OrderStatus 열거형을 사용하여 복잡한 if-else 구문 단순화
- refactor: UserRepository의 사용자 필터링 로직을 스트림 API 사용하여 개선
- refactor: ProductService 전반의 변수명과 메서드명을 더 명확하고 설명적으로 변경
- refactor: OrderService와 PaymentService 간의 순환 의존성 제거를 위한 구조 개선
- refactor: Address 값 객체를 불변 클래스로 변경


`perf`: 성능을 향상시키는 코드 변경   
(캐시 적용 같은 경우 feat로 볼 수도 있습니다. 목적에 따라 선택하면 되지만, 개인적으로 캐시와 같이 기능을 도입한 경우 feat를 사용할 예정입니다. 기능을 도입한 시점에서 버그가 발생할 수도 있어 나중에 도입 지점을 찾기 수월할 것이라 생각하기 떄문입니다)
- perf: UserRepository의 페이징 쿼리에 인덱스 추가로 조회 속도 개선
- perf: OrderService의 주문 목록 조회 시 N+1 문제 해결을 위한 fetch join 적용
- perf: OrderEntity의 불필요한 즉시 로딩을 지연 로딩으로 변경하여 초기 로딩 속도 개선
- perf: UserPointService의 포인트 업데이트를 개별 업데이트에서 벌크 업데이트로 변경
- perf: OrderEntity의 @BatchSize 어노테이션 적용으로 연관 엔티티 조회 쿼리 수 감소

`test`: 테스트 추가, 테스트 코드 리팩토링 또는 기존 테스트 수정(테스트 코드 빌드 또는 리팩토링시, build, refactor를 사용할 수도 있지만 test 관련에서는 test만 사용하는 것이 관리가 편할거라 생각합니다)
- test: UserService의 createUser 메소드에 대한 단위 테스트 추가
- test: OrderController의 주문 생성 API에 대한 통합 테스트 구현
- test: PaymentService의 환불 처리에 대한 엣지 케이스 테스트 추가
- test: UserRepository의 대량 사용자 조회 성능 테스트 구현
- test: ProductServiceTest의 중복 코드를 @BeforeEach 메소드로 추출
- test: 주문 테스트를 위한 OrderTestFixture 클래스 구현
- test: 테스트용 H2 인메모리 데이터베이스 설정 추가

## `<description>`

설명은 변경 사항에 대한 간단한 요약입니다.  
[optional scope] 콜론+공백(`: `) 뒤에 설명이 와야 합니다.   
설명에 대해 구체적이고 자세히 제공하고 싶다면 [optional body]를 이용해야 합니다.   
- 50자 이내로 작성
- 현재 시제 사용 (예: "change"는 사용하고 "changed"나 "changes"는 사용하지 않음)
- 문장 끝에 마침표(.)를 사용하지 않음
- 

## `[optional body]`


설명 다음 빈 줄 하나를 두고 시작합니다.(sourcetree와 같은 경우 나뉘어져 있음)   
자유 형식이고, 개행으로 단락을 구분지을 수 있습니다.   
- 어떻게(How) 변경했는지보다 무엇을(What)과 왜(Why) 변경했는지에 중점
- 72자마다 줄 바꿈(필수는 아니지만 너무 긴 단락은 읽기 힘듬)

## `[optional footer(s)]`


footer로 메타데이터를 포함하는 영역입니다. 주로 Breaking Changes나 이슈 참조에 사용됩니다.   
- 한 줄에 하나의 메타데이터 정보
- 여러 줄의 푸터를 사용할 수 있음
- 빈 줄로 body와 구분(선택이지만, 구분하는 것이 가독성이 좋음)
- `:`, `#` 들과 함께 문자열로 구성되어야 함
- footer 토큰에는 ` ` 대신 `-` 사용하여 구분(본문과 구별하기 위함, BREAKING CHANGE 예외)
- 토큰/구분자(`:`, `#`)로 이루어지고 종료되어야 함
- 주요 변경 사항이 포함된 커밋은 `!`이 type/scope 앞에 포함되어야 함, `BREAKING CHANGE`는 선택
- 대소문자 구별 X, 단 `BREAKING CHANGE`는 대문자여야 함
- 'BREAKING-CHANGE'와 'BREAKING CHANGE` 는 동일 취급합니다.('BREAKING-CHANGE'가 푸터의 일관성 있는 규칙이지만, 'BREAKING CHANGE' 가 일반적인 형태인 듯 싶습니다. 본문에서 'BREAKING CHANGE'로 쓰임)
### footer 사용 예시
```
# 단일 이슈 참조
feat(user): 사용자 프로필 이미지 업로드 기능 추가

사용자가 자신의 프로필 이미지를 업로드할 수 있는 기능을 구현했습니다.

Closes #123
```

```
# 다중 이슈 참조
fix(auth): OAuth 토큰 갱신 오류 수정

OAuth 인증 후 토큰 갱신 과정에서 발생하던 오류를 수정했습니다.

Fixes #456, #789
Related to #234
```

```
# Breaking Changes 명시
feat(api)!: 사용자 API 응답 형식 변경

사용자 정보 조회 API의 응답 형식을 변경했습니다.

BREAKING CHANGE: 'name' 필드가 'firstName'과 'lastName'으로 분리되었습니다.
이전: { "name": "John Doe" }
이후: { "firstName": "John", "lastName": "Doe" }
```

```
docs(api): 결제 API 문서 업데이트

결제 프로세스 API 문서를 최신 버전으로 업데이트했습니다.

Doc-Type: OpenAPI 3.0
Affects: /api/v2/payments
```

## 왜 Conventional Commits를 사용해야 하나?

- `CHANGELOGs` 라는 것을 생성할 수 있습니다.   
  - [github-changelog-generator](https://github.com/github-changelog-generator/github-changelog-generator)
  - [spring-io/github-changelog-generator](https://github.com/spring-io/github-changelog-generator)
- CHANGELOGs를 사용하면 다음 버전 번호를 자동으로 결정해줍니다.(의미있는 버전 관리)
  - fix: 커밋은 PATCH 버전을 증가 (예: 1.0.0 → 1.0.1)
  - feat: 커밋은 MINOR 버전을 증가 (예: 1.0.1 → 1.1.0)
  - BREAKING CHANGE: 포함 커밋은 MAJOR 버전을 증가 (예: 1.1.0 → 2.0.0)
- 구조화된 커밋 메시지를 통해 변경사항의 성격과 목적을 명확히 전달
  - 많이 알려진 커밋 메시지 작성 방식이라 팀원들이 쉽게 이해할 수 있습니다.
- CI/CD 파이프라인에 커밋 메시지를 파싱하여 적절한 작업을 트리거할 수 있습니다.
  - feat: 커밋 → 전체 테스트 실행 후 스테이징 환경에 배포
  - fix: 커밋 → 관련 모듈 테스트 후 패치 배포
  - docs: 커밋 → 문서 사이트 자동 업데이트
- 구조화된 커밋 히스토리로 프로젝트에 쉽게 기여할 수 있도록 합니다.
  - 새로운 사람이 적응하기 용이




## 참조

- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)

