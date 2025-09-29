# Tell, Don't Ask 원칙 깊이 파고들기

## 목차

1.  **도입: 왜 '묻지 말고 시켜야' 할까?**
2.  **핵심 비교: '묻는(Ask)' 코드 vs '시키는(Tell)' 코드**
3.  **구체적인 적용: 어떻게 리팩토링할 것인가?**
4.  **효과와 장점: 무엇을 얻을 수 있는가?**
5.  **주의할 점: 맹목적인 규칙이 아닌, 유용한 지침**
6.  **개념 확장: 연관 원칙과 디자인 패턴**
7.  **결론: 내 코드를 위한 실천 가이드**

---

### 1. 도입: 왜 '묻지 말고 시켜야' 할까?

'Tell, Don't Ask'는 객체에게 **"필요한 데이터를 물어본 후 직접 처리하지 말고, 그냥 해달라고 시켜라"**는 의미를 담고 있는 객체 지향 설계 원칙입니다. 이는 객체가 자신의 데이터와 그 데이터를 처리하는 행동(로직)을 함께 가지고 있어야 한다는 객체 지향의 가장 기본적인 철학을 상기시켜 줍니다.

#### 이 원칙이 해결하고자 하는 문제: 책임의 누수

만약 우리가 이 원칙을 무시하고 객체에게 데이터를 계속 '물어본다면(Ask)' 어떤 문제가 발생할까요?

- **결합도 증가와 OCP 위반:** 데이터를 물어보는 클라이언트 코드는 그 객체의 내부 구조를 속속들이 알게 됩니다. 이는 두 객체 간의 결합도를 높여, 데이터를 가진 객체의 내부 구현이 조금만 바뀌어도 데이터를 물어보던 클라이언트 코드 전체가 수정되어야 하는 연쇄 반응을 일으킵니다.

- **로직 중복과 응집도 저하:** 여러 클라이언트가 동일한 데이터를 물어본 뒤 비슷한 로직을 각자 구현하게 될 가능성이 높습니다. 이는 코드 중복으로 이어지며, 데이터와 그 데이터를 처리하는 로직이 여러 곳에 흩어져 시스템 전체의 응집도를 떨어뜨립니다.

결국, 객체가 스스로 책임져야 할 행동이 외부로 누수되어 시스템 전체의 유지보수성을 악화시키는 것입니다.

#### 해결책: 객체의 자율성과 무결성 보장

'Tell, Don't Ask' 원칙은 바로 이 문제에 대한 해결책으로, 객체에게 **자율성(Autonomy)**을 부여하여 스스로 **무결성(Integrity)**을 지키도록 만듭니다.

> **비유: 전문 바리스타와 손님**
> 손님(클라이언트)은 바리스타(객체)에게 "카페라떼 한 잔 주세요"라고 **'시킬(Tell)'** 뿐, 원두가 몇 그램인지, 우유 온도가 몇 도인지 **'묻지(Ask)'** 않습니다. 레시피와 제조 기술에 대한 모든 결정권, 즉 **자율성**은 바리스타에게 있습니다. 이 자율성을 바탕으로 바리스타는 항상 '맛있고 올바른 카페라떼'라는 결과물의 **무결성**을 스스로 책임지고 보장합니다.

> **[1장 핵심 요약]**
> "'Tell, Don't Ask'는 객체에게 데이터 처리의 **자율성**을 부여하여, 외부의 간섭 없이 스스로 데이터의 **무결성**을 책임지도록 만드는 설계 지침이다."

### 2. 핵심 비교: '묻는(Ask)' 코드 vs '시키는(Tell)' 코드

#### '묻는(Ask)' 코드: 외부에서 모든 것을 결정

```java
// 데이터만 가진 객체 (Anemic Domain Model)
public class CpuMonitor {
    private int value;
    public int getValue() { return value; }
    public void setValue(int value) { this.value = value; }
}

public class Client {
    public void alertService(List<CpuMonitor> monitors) {
        for (CpuMonitor monitor : monitors) {
            // 1. 모니터의 상태를 '묻고(Ask)'
            if (monitor.getValue() > 90) { // 2. 외부에서 직접 판단하고 결정
                System.out.println("경고: CPU 사용량이 90%를 초과했습니다!");
            }
        }
    }
}
```

**분석:** `Client`가 `CpuMonitor`의 내부 상태를 직접 가져와 '90'이라는 비즈니스 규칙과 비교하고, 경고를 보낼지 여부까지 **모두 직접 결정**합니다. `CpuMonitor`는 단순한 데이터 저장소 역할에 그치며, 자신의 책임을 `Client`에게 **전가**하고 있습니다.

#### '시키는(Tell)' 코드: 내부에 책임을 위임

```java
import java.util.function.Consumer;

// 데이터와 행동을 함께 가진 객체
public class CpuMonitor {
    private int value;
    private final int alertThreshold;
    private final Consumer<CpuMonitor> alertAction;

    public CpuMonitor(int alertThreshold, Consumer<CpuMonitor> alertAction) {
        this.alertThreshold = alertThreshold;
        this.alertAction = alertAction;
    }

    public void setValue(int value) { this.value = value; }

    // 외부에서는 이 메서드를 '호출(Tell)'하기만 하면 됨
    public void sample() {
        if (this.value > this.alertThreshold) { // 1. 스스로 상태를 판단하고
            this.alertAction.accept(this);      // 2. 주입받은 행동을 실행
        }
    }
}

public class Client {
    public void alertService(List<CpuMonitor> monitors) {
        for (CpuMonitor monitor : monitors) {
            // 모니터에게 '샘플링하라'고 시킬(Tell) 뿐, 내부 로직은 전혀 모름
            monitor.sample();
        }
    }
}
```

**분석:** `Client`는 더 이상 `CpuMonitor`의 상태를 묻지 않고, `sample()`이라는 메시지를 보내 **'할 일을 하라'고 시킬(Tell) 뿐**입니다. 모든 책임이 `CpuMonitor` 내부로 **위임**되었고, 두 객체 간의 결합도는 극적으로 낮아졌습니다.

> **[2장 핵심 비유]** > **묻는(Ask) 코드**로 작업하는 것은, 모든 부품(데이터)을 배송받아 직접 조립(로직 수행)하는 '조립식 가구'와 같다. 모든 책임은 조립하는 사람에게 있다. 반면, **시키는(Tell) 코드**는 전문가가 완벽하게 조립한 '완성품 가구'를 주문하는 것과 같다. 우리는 그저 "이 자리에 놓아주세요"라고 '시키기'만 하면 된다.

### 3. 구체적인 적용: 어떻게 리팩토링할 것인가?

'Tell, Don't Ask' 리팩토링의 핵심은 **"이 데이터를 가지고 궁극적으로 무엇을 하려고 하는가?"** 라는 질문을 던져 최종 목표를 파악하고, 그 목표를 수행할 책임을 가장 적절한 객체에게 위임하는 것입니다.

#### 3.1. 기본 원칙 적용: Customer와 Wallet 예시

**Before (Ask):** `PayBillService`가 손님(`Customer`)의 지갑(`Wallet`)을 마음대로 열어보고 돈을 꺼내 갑니다.

```java
class PayBillService {
    public void handle(Customer customer, int paymentAmount) {
        Wallet wallet = customer.getWallet();
        int currentCash = wallet.getCash();
        if (currentCash >= paymentAmount) {
            wallet.setCash(currentCash - paymentAmount);
        }
    }
}
```

**After (Tell):** `PayBillService`는 `Customer`에게 `pay`하라고 '시킬' 뿐입니다. `Wallet`은 스스로 잔액을 확인하여 자신의 무결성을 책임집니다.

```java
class PayBillService {
    public void handle(Customer customer, int paymentAmount) {
        customer.pay(paymentAmount); // "지불하세요" 라고 '시킬' 뿐이다.
    }
}

class Customer {
    private Wallet wallet;
    public void pay(int amount) {
        this.wallet.withdraw(amount);
    }
}
```

#### 3.2. 심화 적용: Order와 배송비 계산 예시

**Before (Ask):** '기차 충돌(Train Wreck)' 코드로, `Client`가 `Order`의 속사정을 모두 캐물어 배송비를 계산합니다.

```java
class Client {
    public int calculateShippingFee(Order order) {
        String city = order.getMember().getAddress().getCity();
        if ("서울".equals(city)) { return 2500; }
        return 3000;
    }
}
```

**After (Tell):** `Client`는 `Order`에게 배송비를 계산하라고 '시키기'만 합니다. '배송비 계산' 책임이 전문가인 `Order` 객체에게 위임되었습니다.

```java
class Client {
    public int calculateShippingFee(Order order) {
        return order.calculateShippingFee();
    }
}

class Order {
    private Member member;
    public int calculateShippingFee() {
        String city = member.getCityOfAddress();
        if ("서울".equals(city)) { return 2500; }
        return 3000;
    }
}
```

> **[3장 핵심 요약 및 비유]**
>
> - **요약:** "'묻지 않고 시키기' 리팩토링의 핵심은, 코드의 '궁극적인 목표'를 파악하여 그 목표를 수행할 **책임을 가장 적절한 객체에게 옮겨주는 과정**이다."
> - **비유:** 리팩토링은 여러 부서(`Member`, `Address`)를 돌아다니며 보고서(`배송비 계산`)를 직접 만드는 대신, **보고서 작성 책임을 가진 기획팀(`Order`)**에 일임하는 것과 같다.

### 4. 효과와 장점: 무엇을 얻을 수 있는가?

'Tell, Don't Ask' 원칙의 모든 이점은 **'캡슐화 강화'** 에서 시작되어 다음과 같은 선순환 효과를 낳습니다.

1.  **캡슐화 강화:** 객체는 외부의 간섭 없이 스스로를 책임지는 **'자율적인 존재'**가 되어 스스로의 **무결성**을 보장할 수 있게 됩니다.
2.  **응집도 향상:** 특정 데이터와 관련된 비즈니스 로직이 데이터를 소유한 객체 안으로 모여, **관련된 모든 것이 한 곳에 있게 됩니다.** 수정 지점이 명확해지고 코드 파악이 쉬워집니다.
3.  **결합도 감소:** 클라이언트는 협력 객체의 내부 구조를 알 필요 없이 공개된 메서드에만 의존하게 됩니다. 이로 인해 **코드 중복이 자연스럽게 제거**되고, 변경에 유연한 구조가 만들어집니다.

> **[4장 핵심 비유]**
> 이 원칙은 **마이크로서비스 아키텍처(MSA)** 철학과 닮아있다. 각 서비스(객체)는 높은 **응집도**를 갖고, 다른 서비스와는 잘 정의된 API를 통해 **'시키기(Tell)'**만 한다. 이 API 덕분에 서비스 간 **결합도**가 낮아져, 각 서비스를 독립적으로 개발하고 배포할 수 있게 된다.

### 5. 주의할 점: 맹목적인 규칙이 아닌, 유용한 지침

'원칙(Principle)'이라는 단어 때문에 'Tell, Don't Ask'를 반드시 지켜야 할 엄격한 **규칙(Rule)**으로 오해하기 쉽습니다. 하지만 이 원칙은 **지침(Guideline)**에 더 가깝습니다. 지침이란, 더 나은 설계를 위해 '한번쯤 이렇게 생각해보라'고 방향을 제시해주는 유용한 조언과 같습니다. 모든 상황에 맹목적으로 적용하기보다, 그 본질을 이해하고 상황에 맞게 사용하는 지혜가 필요합니다.

- **모든 '묻기(Ask)'가 나쁜 것은 아니다:** DTO나 UI 렌더링처럼, 데이터를 '묻는' 행위가 비즈니스 '결정'으로 이어지지 않고 단순히 조회/전송이 목적일 경우, `getter` 사용은 합리적이고 효율적인 설계입니다.
- **'시키는(Tell)' 것의 부작용:** 한 객체에 너무 많은 책임을 위임하면 '거대 객체(God Object)'가 되어 단일 책임 원칙(SRP)을 위반할 수 있습니다. 이때는 'Tell, then Delegate' 방식으로, 받은 명령을 다시 전문화된 다른 객체에게 위임하며 책임을 분산시켜야 합니다.

> **[5장 핵심 요약 및 비유]**
>
> - **요약:** "'Tell, Don't Ask'는 더 나은 설계를 위한 유용한 **'질문'**이지 모든 상황에 대한 **'정답'**이 아니므로, 코드의 복잡도와 다른 설계 원칙과의 균형을 항상 고려해야 한다."
> - **비유:** 이 원칙은 '비타민'과 같은 **지침**이다. 적절히 섭취하면 몸(코드)을 건강하게 만들지만, 이것만 맹신하여 부작용(거대 객체)을 일으켜서는 안 된다. 중요한 것은 전체적인 식단(아키텍처)의 균형을 고려하는 의사(개발자)의 전문적인 진단이다.

### 6. 개념 확장: 연관 원칙과 디자인 패턴

'Tell, Don't Ask'는 다른 원칙 및 패턴들과 깊은 관계를 맺고 서로를 강화합니다.

- **디미터 법칙 (Law of Demeter):** '기차 충돌' 코드를 방지하여 **'누구에게 말할 것인가'**를 제한합니다. TDA는 **'무엇을 말할 것인가'**를 제한하여 디미터 법칙을 보완합니다.
- **정보 은닉 (Information Hiding):** TDA는 객체 내부 로직을 숨김으로써 **정보 은닉을 실현하는 가장 강력한 실천법**입니다.
- **전략 패턴 (Strategy Pattern):** 복잡한 'Tell' 로직을 별도의 객체로 캡슐화하여, 책임을 **위임하며 시킬 수 있게** 해줍니다.
- **옵저버 패턴 (Observer Pattern):** '도메인 이벤트'를 통해 객체 간 직접적인 결합 없이, **느슨하게 연결된 'Tell'**을 가능하게 합니다.

> **[6장 핵심 요약]**
> 'Tell, Don't Ask'를 중심으로 보면, 각 개념은 다음과 같은 역할을 수행한다.
>
> - **근본 철학 (Why):** **정보 은닉** 원칙을 지키기 위한 구체적인 실천법이다.
> - **적용 신호 (When):** **디미터 법칙**을 위반하는 코드는 'Tell, Don't Ask'를 적용할 좋은 후보다.
> - **구현 방법 (How):** **전략 패턴**은 복잡한 'Tell'을 위임하는 방법이며, **옵저버 패턴**은 느슨하게 결합된 'Tell'을 구현하는 방법이다.

### 7. 결론: 내 코드를 위한 실천 가이드

#### 7.1. 'Tell, Don't Ask'를 적용하기 좋은 신호 찾기

1.  **`get`으로 시작해서 `if`로 끝나는 로직:** 결정 로직을 데이터를 가진 객체 내부로 옮길 것.
2.  **'기차 충돌(Train Wreck)' 코드 (`.get().get()`):** 궁극적인 목표를 파악하여 시작 객체에 위임할 것.
3.  **비대한 서비스와 빈약한 도메인 객체:** 서비스의 로직을 도메인 객체의 '행위'로 옮길 것.

#### 7.2. 핵심 원칙 요약 및 정리

'Tell, Don't Ask'의 여정을 한 문장으로 요약하자면 다음과 같습니다.

> **객체에게 상태를 묻고 외부에서 결정하지 말고, 객체에게 원하는 것을 시켜 스스로 결정하고 행동하게 하라.**

기억해야 할 것은, 'Tell, Don't Ask'는 엄격한 규칙이 아니라, 더 나은 객체 지향 설계를 위한 끊임없는 **질문**이자 **지침**이라는 사실입니다. 이 지침을 나침반 삼아, 우리는 책임이 올바르게 분배된, 살아 숨 쉬는 객체들의 공동체를 만들어나갈 수 있을 것입니다.

---

### 참조

- [Tell Dont Ask - Martin Fowler](https://martinfowler.com/bliki/TellDontAsk.html)

- [Understanding the TDA(Tell Don't Ask) Principle - shiiyan](https://medium.com/@shiiyan/understanding-the-tda-telldontask-principle-44acbc28bacb)

- [Tell, Don't Ask! - Ardalis Youtube](https://www.youtube.com/watch?v=AcYcbBVmZew)

- [Tell, Don't Ask - DevIQ](https://deviq.com/principles/tell-dont-ask)

- [Tell, don't ask 원칙(TDA 원칙) - Effective Programming](https://effectiveprogramming.tistory.com/entry/Tell-dont-ask)
