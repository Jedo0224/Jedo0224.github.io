---
title:  "싱글톤 컨테이너"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Spring
tags:
  - [Spring, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-04-13
last_modified_at: 2024-04-14
---


## 강의 링크
[스프링 핵심 원리 - 기본편 강의 - 인프런](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)

<br/>


## 웹 어플리케이션과 싱글톤

- 웹 어플리케이션은 보통 여러 고객이 동시에 요청을 한다.
    
    appconfig는 고객이 요청할 때마다 new로 객체를 만들면 메모리 소모가 많아지게 된다.
    
![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/b95ef044-4c48-46df-ad4c-ebed41e07531)


```java
  @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();
        //1. 조회 : 호출 할 때마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();
        MemberService memberService2 = appConfig.memberService();

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        //memberService1 != memberService2
        Assertions.assertNotEquals(memberService1,memberService2);
    }
```

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/7b1f5d7f-ae30-498a-bfec-f4817f0d55d6)


- **assert - 주장하다**
- 이 경우 memberService안에 있는 MemoryMemberRepository도 같이 생성하게되어 메모리 소요량이 과중된다.

→ 해결 방안은 해당 객체를 딱 하나만 생성하고, 공유하도록 설계하면 된다. → **“싱글톤 패턴”** 

<br/>
<br/>
<br/>
<br/>

## 싱글톤 패턴

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 규제해야한다.
    - private를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막는다!

```java

public class SingletonService {
    //자기 자신을 내부에 private static으로 선언하면 클래스에 딱하나만 존재하도록 변경됨.

    //1, static 영역에 객체를 딱 1개만 생성해둔다.
    private static final SingletonService instance = new SingletonService();

    //2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 한다.
    public static SingletonService getInstance(){
        return instance;
    }

    //3. 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
    private SingletonService() {
    }

    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }

```

1. static 영역에 객체 instance를 미리 하나 만들어놓는다.
2. 이 객체 인스턴스를 조회하려면 오직 getInstance()를 통해서만 조회할 수 있다.
3. 딱 1개의 객체 인스턴스가 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new 키워드로 객체 인스턴스가 생성되는 것을 막는다.

<br/>
<br/>

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/2342d9fb-fb21-4f4c-b126-2045b1cedde4)

```java
@Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest() {
        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();

        System.out.println("singletonService1 = " + singletonService1);
        System.out.println("singletonService2 = " + singletonService2);

        Assertions.assertSame(singletonService1,singletonService2);
        // same == 참조값
        // equal -> java equals (객체의 )
    }
```

- 싱글톤 패턴의 문제점
    1. 싱글톤 패턴을 구현해야함
    2. 의존 관계상 클라이언트가 구체 클래스에 의존한다. (DIP 위반)
        * 구체 클래스의 .getinstance등을 사용해야함.
    3. 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
    4. 테스트가 어렵다.
    5. 내부 속성을 변경하거나 초기화 하기 어렵다.
    6. private 생성자로 자식 클래스를 만들기 어렵다.
    7. 결론적으로 유연성이 떨어진다.
    8. 안티패턴으로 불리기도 한다.

### 그럼 AppConfig에 사용되는 Class들을 모두 private로 변경해야하나?

spring을 사용하면 자동으로 객체를 싱글톤으로 사용해주기에 매우 편리하다. ( 싱글톤 패턴의 문제점을 해결한 싱글톤 컨테이너 사용 ) 

<br/>
<br/>
<br/>
<br/>

## 싱글톤 컨테이너

- 싱글톤 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
    
    ![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/521fa031-7af8-49b4-a38a-60e3d0b69720)

 
    - 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 싱글톤으로 등록한다.
    
- 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
    - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
    - DIP, OCP, 테스트, private 생자 부터 자유롭게 싱글톤을 사용할 수 있다.

```java
 @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
//        AppConfig appConfig = new AppConfig();
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        //memberService1 != memberService2
        Assertions.assertSame(memberService1,memberService2);

    }
```

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/22963123-1a88-420a-8266-a3a473a7af87)


![Untitled 4](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/b0048d29-6a9d-4fbd-9135-d6ea19dd6012)

- 스프링 컨테이너 덕분에 객체를 공유해서 효율적으로 재사용할 수 있다.
- 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니다. 요청할 때 마다 새로운 객체를 생성해서 반환하는 기능도 제공한다 → **빈 스코프**

<br/>
<br/>
<br/>
<br/>

## 싱글톤 방식의 주의점

- 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체를 공유하기 때문에 싱글톤 객체는 “무상태”(stateless)로 설계해야한다.
    - 무상태(stateless)
        - 특정 클라이언트에 의존적인 필드가 존재하면 안된다.
        - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
        - 가급적 조회 및 읽기만 가능해야한다.
        - 필드 대신에 공유되지 않는 , 지역변수, 파라미터 , TreadLocal 등을 사용해야한다.
            - ThreadLocal : 각 스레드가 독립적으로 초기화된 자신만의 변수 복사본을 가지고 있다. 이처럼 각 스레드가 변수를 독립적으로 사용할 수 있으며, 다른 스레드와 공유하지 않는다.
    
    ctrl + shit + t : 테스트 클래스 생성.
    
    ### StatefulService
    
    ```java
    public class StatefulService {
    
        private int price;
    
        public void order(String name, int price){
            System.out.println("name = " + name + "price = " + price);
            this.price = price;
        }
    
        public int getPrice(){
            return price;
    
        }
    }
    ```
    
    ```java
        @Test
        void statefulServiceSingleton(){
            ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
            StatefulService statefulService1 = ac.getBean(StatefulService.class);
            StatefulService statefulService2 = ac.getBean(StatefulService.class);
    
            statefulService1.order("userA",10000);
            statefulService2.order("userB",20000);
    
            int price = statefulService1.getPrice();
            System.out.println(price);
        }
    ```
    
    ![single](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/4cce36cb-99d2-4ec9-904a-2ad65af46197)



    사용자 A는 10000원을 주문했는데 20000원이 주문될 수 있음. (**위험!**)
    
    - 싱글톤 컨테이너가 되어 공유 필드가 형성되는 경우가 있다. → 항상 조심해야한다
    - 스프링 빈은 항상 무상태(stateless)로 설계해야한다.
      

    
    ### StatelessService
    
    ```java
    public class StatelessService {
    
        public int order(String name, int price){
            System.out.println("name = " + name + "price = " + price);
            return price;
        }
    }
    ```
    
    ```java
        @Test
        void statelessServiceSingleton(){
            ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
            StatefulService statefulService1 = ac.getBean(StatefulService.class);
            StatefulService statefulService2 = ac.getBean(StatefulService.class);
    
            int userA = statefulService1.order("userA",10000);
            int userB = statefulService2.order("userB",20000);
    
            System.out.println("userA price = " + userA);
            Assertions.assertEquals(userA,userB);
        }
    ```
    
    ![Untitled 6](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/81743830-2ae2-4b02-b2cc-36db8434c2a1)

    <br/>
    <br/>
    <br/>
    <br/>
    

## Configuration과 싱글톤

- 과연 스프링 컨테이너는 각 빈에 등록해놓은 MemoryMemberRepository 객체를 몇번 선언할까?
    - 3번?(MemberServiceImpl + orderService + memberRepository())
    - 1번?

```java
@Configuration // 설정 정보 , 구성 정보
public class AppConfig {

    @Bean // 스프링 컨테이너에 등록된다.
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(
                memberRepository(),discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}

```

```java
public class ConfigurationSingletonTest {
    @Test
    void configurationTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        //모두 같은 리포지토리 참조값을 반환함.
        System.out.println("memberService -> memberRepository = " + memberService.getMemberRepository());
        System.out.println("orderService -> memberRepository = " + orderService.getMemberRepository());
        System.out.println("memberRepository = " + memberRepository);

    }
}

```

![Untitled 7](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/0172fe05-c769-4541-90fb-4326215a0889)


- 모두 같은 인스턴스가 조회되고 있는 것을 확인할 수 있다.

    <br/>
    <br/>

### Trouble shooting

```java
@Configuration // 설정 정보 , 구성 정보
public class AppConfig {

    @Bean // 스프링 컨테이너에 등록된다.
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public static MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(
                memberRepository(),discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}

```

![Untitled 8](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/a4dd6989-29d0-49e1-bf85-23f4e1176736)

- public static MemberRepository로 선언할 경우 3개의 MemberRepository 모두 다른 값이 참조됨.

**static 메소드의 영향**

- **`static`** 메소드는 클래스 레벨에서 실행되므로 인스턴스 레벨의 프록시 처리가 적용되지 않는다.
    - 프록시 (proxy)
     객체를 대신해서 특정 작업을 수행하는 객체 → CGLIB 라이브러리
- **따라서 `@Bean`이 붙은 `static` 메소드는 스프링의 프록시 메커니즘에 의해 관리되지 않는다.**

```java
 @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
```


<br/>
<br/>
<br/>
<br/>


## @Configuration과 바이트코드 조작의 마법

- MemoryMemberRepository를 어떻게 1번만 선언해서 사용하는 것일까?

```java
 @Test
    void configurationDeep(){  ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);
        System.out.println("bean = " + bean+getClass());
    }
```

![Untitled 9](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/7d46d6d8-f03c-41ff-904f-a657256124b3)

- 참조값이 복잡해진것을 볼 수 있다.
    - CGLIB라는 바이트코드 조작 라이브러리를 사용한다.(Code Generator Library)
        1. AppConfig 클래스를 상속받아 임의의 다른 클래스를 만든다.
        2. 그 클래스를 스프링 빈에 등록한다.
        
        ![Untitled 10](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/73638186-de0b-45e6-a526-3e2b4ebdb16f)

        
- 스프링 컨테이너에 해당 스프링 빈이 있으면 이미 있는 것을 사용.
- 없을 경우는 새롭게 빈을 등록해서 사용.
    
    → 이를 통해 싱글톤을 유지한다.
    

### 만약 @Configuration이 없다면?

- @Bean만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않는다.
