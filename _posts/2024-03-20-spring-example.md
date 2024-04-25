---
title:  "스프링 핵심 원리 이해1 - 예제 만들기 (1)"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Spring
tags:
  - [Spring, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-03-19
last_modified_at: 2024-03-20
---


## 강의 링크
[스프링 핵심 원리 - 기본편 강의 - 인프런](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)

## 예제 만들기

- 비즈니스 요구사항과 설계
    - 회원
    1. 회원을 가입하고 조회할 수 있다.
    2. 회원은 일반(Basic)과 VIP 두 가지 등급이 있다.
    **회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)**
    - 주문과 할인 정책
    1. 회원은 상품을 주문할 수 있다.
    2. 회원 등급에 따라 할인 정책을 적용할 수 있다.
    3. 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
    **할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)**

- 회원 클래스 다이어그램 / 회원 객체 다이어그램
    - 회원 클래스 다이어그램
        - 실제 서버를 실행하지 않고 클래스만을 분석해서 보여주는 다이어그램.
    - 회원 객체 다이어그램
        - 실제 서버가 뜰 때 new를 이용하여 생성되는 실제 인스턴스 끼리의 참조.
     
      
![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/9eaea1b3-cc10-44e7-96fb-80bf4767078b)

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/1edac002-db0f-4fcb-9885-56c539985d43)

### 실습 코드 설계의 문제점

```java
package hello.core.member;

public class MemberServieImpl implements MemberService{
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {

        return memberRepository.findById(memberId);
    }
}
```

- 다른 저장소로 변경할 때 OCP 원칙을 준수 할 수 없음.
- DIP를 준수하고 있지 않음.
    - “의존관계가 잍너페이스 뿐만 아니라 구현까지 모두 의존하고 있기 때문에”
    

## 주문 - 할인과 도메인 개발

### 역할

1. 주문 생성: 클라이언트는 주문 서비스에 주문 생성을 요청한다.
2. 회원 조회: 할인을 위해서는 회원 등급이 필요하다. 그래서 주문 서비스는 회원 저장소에서 회원을 조회한다.
3. 할인 적용: 주문 서비스는 회원 등급에 따른 할인 여부를 할인 정책에 위임한다.
4. 주문 결과 반환: 주문 서비스는 할인 결과를 포함한 주문 결과를 반환한다

![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/a3716cf3-d2c5-4f04-ba8c-780b2bb81a44)


![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/93c3ac2c-7eba-49f7-adcd-cece72ec1a47)

```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member,itemPrice);
        return new Order(memberId,itemName,itemPrice,discountPrice);
    }
}
```

### 잘 설계한 코드인 이유

OrderServiceImpl 는 discountPolicy의 역할 및 책임을 알 필요 없이 값만 넘겨주어도 자신에  역할을 수행하는 것에 있어서 영향 받는 것이 없다. → 단일 책임 원칙이 잘 설계되었다고 볼 수 있다.
