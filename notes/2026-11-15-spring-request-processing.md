# Spring 요청 처리: 공유 상태, DI, 역할 분리와 트랜잭션

- 날짜: 2026-11-15 (Asia/Seoul)
- 수업: CS-06 / Spring
- 진행: 확인 문제 5/5개 답변·피드백 완료. 정답 5개나 숙련 완료를 의미하지 않는다.
- 같은 날 Java 수업의 마지막 1문제를 마친 뒤 별도로 진행한 5문제다.
- 목표: 공유 객체와 호출별 작업 공간, 의존 객체 연결, 요청 처리 역할, 하나의 업무를 묶는 트랜잭션 경계를 설명하기.
- 예제는 일반적인 학습 사례이며 실제 서비스의 구현·테스트 결과가 아니다.

## 핵심 개념

| 개념 | 설명 |
|---|---|
| 빈과 컨테이너 | 빈은 Spring이 관리하는 객체, 컨테이너는 빈의 생성·관리·연결을 담당 |
| 기본 싱글톤 범위 | 한 컨테이너 안에서 같은 빈 정의에 대해 객체 하나를 공유. 프로그램 전체의 모든 같은 클래스 객체가 하나라는 뜻은 아님 |
| 객체 필드 | 같은 객체를 사용하는 호출들이 같은 필드에 접근할 수 있음. 싱글톤이라는 이유만으로 요청별 변경이 안전해지지 않음 |
| 호출별 프레임 | 같은 메서드도 호출마다 매개변수·지역 변수 등을 위한 별도 작업 공간이 생김. 메서드 코드 자체가 복제되는 것은 아님 |
| DI | Dependency Injection, 필요한 의존 객체를 외부에서 전달받는 방식. 생성자로 받는 형태가 생성자 주입 |
| 역할 분리 | 이 예제는 Controller가 HTTP 처리, Service가 업무 규칙·순서, Repository가 DB 접근을 담당하는 구조를 사용 |
| 트랜잭션 경계 | 어느 작업들을 함께 확정하거나 취소할지 정하는 범위 |

역할 분리는 예제의 설계이며 Spring이 모든 프로그램에 세 계층을 강제한다는 뜻이 아니다. 지역 참조 변수가 따로 있어도 같은 객체를 가리킬 수 있으므로 모든 참조 대상까지 서로 독립이라고 일반화하지 않는다.

## 확인 문제와 피드백

### 1. 싱글톤 빈의 공유 필드

**질문:** GreetingService가 한 컨테이너에 기본 싱글톤 빈 하나로 등록돼 있고 두 요청이 같은 객체를 사용한다. A가 greet("A")에서 이름을 저장하고 반환 직전에 멈춘다. B가 greet("B")를 저장·반환한 뒤 A가 재개해 필드를 읽는다. A에게 반환되는 문자열과 이유는? 다른 변경은 없으며 앞선 쓰기가 다음 읽기에 반영되는 예제 조건이다.

```java
class GreetingService {
    private String currentName;

    public String greet(String name) {
        currentName = name;
        return "안녕, " + currentName;
    }
}
```

**사용자 답변 요지:** A에게 안녕 A가 반환될 것 같다. return으로 하나만 나올 수 있으므로 B는 무시될 것 같다. 둘 다 반환된다면 안녕 B 다음 안녕 A 순서일 것 같다.

**피드백:** B 호출이 먼저 끝나고 A 호출이 나중에 끝난다는 순서는 맞다. 하지만 내용은 둘 다 안녕 B다. return은 해당 호출 하나를 끝내며 다른 호출의 반환을 없애지 않는다. 공유 currentName은 B가 덮어썼으므로 A도 B를 읽는다. A가 재개돼도 필드를 A로 자동 복구하거나 대입부터 다시 실행하지 않는다.

| 순서 | 공유 currentName | 결과 |
|---|---|---|
| A가 이름 저장 | A | 아직 반환 안 함 |
| B가 이름 저장 후 반환 | B | B에게 안녕 B |
| A가 재개해 반환 | B | A에게 안녕 B |

주어진 실행 순서의 학습 사례이며 일반 필드의 모든 스레드 간 가시성을 보장한다는 설명은 아니다.

### 2. 공유 필드를 없애고 매개변수 사용

**질문:** 같은 싱글톤 객체에서 필드를 없애고 다음 코드를 사용한다. A가 greet("A")에서 반환 직전에 멈춘 동안 B가 greet("B")를 호출·반환하고 A가 다시 진행한다. B와 A의 반환값 및 서로 영향을 주거나 주지 않는 이유는?

```java
class GreetingService {
    public String greet(String name) {
        return "안녕, " + name;
    }
}
```

**첫 사용자 답변:** B에게 안녕 B, A에게도 안녕 B일 것 같다. 아직 A와 B가 공유 자원을 함께 쓰는 것 같다.

**보충 설명:** 같은 객체와 메서드 코드를 사용해도 호출별 프레임은 다르다. A 호출의 name은 A, B 호출의 name은 B이며 B의 인자 전달이 A의 name을 덮어쓰지 않는다. B가 끝나면 B 프레임만 없어지고 A는 자신의 name으로 안녕 A를 반환한다. 이전 문제의 currentName 필드는 이 코드에 없다.

**같은 문제 안의 재확인:** B가 끝난 뒤 A가 return에서 사용하는 name은 어느 호출의 작업 공간에 있는 값인지 물었다. 사용자는 “함수 단위로 공간이 갈리나?”라며 A 호출의 작업 공간 값을 가져올 것 같다고 답했다.

**피드백:** A의 작업 공간을 선택한 판단은 정확하다. 함수마다 공간이 하나가 아니라 같은 함수도 호출마다 별도 공간이 생긴다고 표현을 보완했다. 전체 반환값은 이미 설명한 뒤이며 사용자는 작업 공간의 소속을 재확인했다. 새로운 사례에서 독립적으로 모두 설명한 것으로 기록하지 않는다. 이 보충은 6번째 문제가 아니라 문제 2 안의 설명이다.

### 3. 생성자 주입과 테스트용 저장소 교체

**질문:** MemberRepository는 회원 조회 기능의 인터페이스이며 DbMemberRepository와 FakeMemberRepository가 이를 구현한다. 실제 DB 없이 가짜 저장소로 테스트하려면 서비스 내부 코드를 바꾸지 않고 교체하기 쉬운 방식은 어느 쪽이며 왜인가?

```java
// 방식 A: 내부에서 구체적인 저장소 생성
class MemberServiceA {
    private final MemberRepository repository = new DbMemberRepository();
}

// 방식 B: 외부에서 생성자로 제공
class MemberServiceB {
    private final MemberRepository repository;

    public MemberServiceB(MemberRepository repository) {
        this.repository = repository;
    }
}
```

질문에서는 두 대안 모두 MemberService라는 이름으로 별도 코드 블록에 제시했다. 위 이름의 A/B 접미사는 기록에서 두 대안을 함께 구분하기 위한 것이다.

**사용자 답변 요지:** B가 낫다. 직접 생성해서 참조하는 것보다 분리된 인터페이스를 가져다 쓰는 것이 결합도 측면에서 좋아 보인다.

**피드백:** 선택과 결합도를 낮추는 방향은 맞다. 두 방식 모두 인터페이스 타입을 사용한다. 차이는 A가 구체적인 DbMemberRepository의 생성까지 결정하지만 B는 외부가 선택한 객체를 받는다는 것이다. 외부에서 new MemberServiceB(new FakeMemberRepository())로 테스트용 구현을 제공할 수 있다. 생성·선택 책임이 외부로 이동한다는 정확한 이유와 코드 예시는 보충 설명이다.

DI 자체는 Spring 없이도 가능하며 Spring은 빈 등록·설정에 맞게 연결을 수행한다. 생성자 코드를 썼다고 빈 등록과 모든 구현 선택이 자동으로 완성된다고 일반화하지 않는다.

### 4. 웹과 배치의 공통 업무 규칙

**질문:** 사용자의 웹 주문은 Controller를 통해 실행하고, 밤마다 예약 주문을 처리하는 배치는 HTTP Controller를 거치지 않는다. 두 경로 모두 재고가 부족하면 주문을 거절해야 한다. Controller·Service·Repository로 역할을 나눈 이 설계에서 공통 규칙과 주문 흐름을 어디에 두어 재사용할 것인가? Controller에만 규칙이 있으면 어떤 문제가 있는가?

**사용자 답변 요지:** Service에 두고 두 경로가 재사용한다. Controller에만 규칙을 두면 맡지 않아도 되는 역할을 줘서 유지보수 시 로직을 찾는 시간이 길어질 것 같다.

**피드백:** Service 선택과 Controller의 책임 증가·유지보수 문제는 타당하다. 더 직접적인 문제는 Controller를 거치지 않는 배치에서 검사가 누락될 수 있다는 점이다. 웹과 배치가 공통 Service를 호출하도록 해 규칙을 함께 적용할 수 있다. 배치의 규칙 우회 가능성은 보충 설명이며 사용자 답변으로 확인했다고 기록하지 않는다. 동시 주문의 재고 잠금은 이 문제에서 다루지 않았다.

### 5. 주문 저장과 재고 차감의 트랜잭션 경계

**질문:** 주문 저장 후 재고 차감을 실행한다. 두 변경 모두 성공하거나 둘 다 취소해야 한다. 주문 저장은 성공했지만 재고 차감이 변경 전에 RuntimeException으로 실패해 Service 밖으로 전파됐다. 방식 A는 각 Repository 호출이 별도 트랜잭션이고 주문 저장 반환 시 이미 커밋한다. 방식 B는 placeOrder 전체를 @Transactional로 묶어 두 호출이 같은 트랜잭션에 참여한다. 실패 후 각 방식의 주문 기록 유지 여부, 규칙을 지키는 방식과 이유는?

```java
public void placeOrder() {
    orderRepository.saveOrder();
    stockRepository.decreaseStock();
}
```

트랜잭션 설정과 호출 경로가 정상 적용되고 같은 DB의 작업을 처리하는 조건이다. 다른 요청·외부 시스템 작업은 없고 예외를 중간에 잡아서 삼키지 않는다. 이번 RuntimeException은 기본 롤백 대상이며 모든 예외가 기본적으로 롤백된다는 뜻은 아니다.

**사용자 답변 요지:** A는 주문은 저장되지만 재고는 차감되지 않는다. B는 전체 롤백돼 주문이 없던 상태로 돌아간다. B가 업무 규칙을 지키며 placeOrder가 실패하면 돌아가야 거래의 일관성이 지켜진다.

**피드백:** 결과와 B 선택이 정확하다. A에서는 이미 확정된 주문을 뒤의 별도 트랜잭션 실패가 되돌리지 못한다. B에서는 아직 확정하지 않은 주문 저장을 함께 취소한다.

부분 반영을 막고 전부 반영 또는 전부 취소하도록 하는 성질은 원자성이다. 이 예제에서는 그 원자성으로 주문·재고의 결합 규칙도 유지하므로 일관성과 연결한 방향은 타당하다. 다만 @Transactional이 모든 업무 규칙을 자동으로 구현하거나 동시성 문제 전반을 해결한다는 뜻은 아니다. 정확한 ACID 용어 구분은 피드백이며 재답변으로 검증하지 않았다.

## 확인된 내용과 다음 복습

- 확인: 문제 2 보충 후 A 호출의 작업 공간 선택, 생성자 주입 선택과 결합도 감소 방향, Service 역할과 유지보수 이유, 트랜잭션별 최종 상태와 업무 단위 전체 롤백 선택.
- 복습 필요: 공유 필드와 호출별 매개변수, return의 호출별 범위, 함수마다가 아닌 호출마다 프레임 생성, 인터페이스 사용과 외부 주입의 차이, 배치의 Controller 우회, 원자성과 일관성의 정확한 구분.
- 보충·미검증: 새로운 코드 사례의 공유 상태 판단, Spring 빈 등록·생성자 선택의 실제 설정, 트랜잭션 프록시·자기 호출·전파 옵션과 checked 예외 처리. 실제 앱 실행·테스트는 하지 않았다.
- 범위: 요청 처리는 Controller·Service·Repository 역할 수준으로 다뤘다. DispatcherServlet 내부 처리, 필터·인터셉터·메시지 변환기 전체를 학습 완료했다고 기록하지 않는다.
- 11월 15일 답변한 문제는 이전 Java 마지막 1개와 이번 Spring 5개, 총 6개다. 수업은 각각 별도로 기록한다.
- 다음 기본 주제: 11월 16일 예정 CS-07 웹 요청의 네트워크 흐름. 총 5문제 안에 필요한 복습을 포함한다.

## 참고 자료

- [Spring Bean Scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html): 기본 싱글톤의 컨테이너·빈 정의별 범위.
- [JVM 프레임과 지역 변수](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-2.html#jvms-2.6): 호출마다 생성되는 프레임과 매개변수.
- [Spring Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html): 생성자 주입과 의존 구현 교체.
- [Spring MVC Controllers](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html), [계층별 컴포넌트](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html): HTTP 처리와 역할 분리의 기초.
- [Using @Transactional](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html), [롤백 규칙](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html): 선언적 트랜잭션과 기본 롤백 조건.
- Spring 자료 확인일: 2026-11-15. JVM 명세는 앞선 Java 학습에서 확인한 자료를 재사용했다.
