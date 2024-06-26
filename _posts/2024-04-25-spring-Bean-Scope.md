---
title:  "빈 스코프"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Spring
tags:
  - [Spring, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-04-24
last_modified_at: 2024-04-25
---

## 강의 링크
[스프링 핵심 원리 - 기본편 강의 - 인프런](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)

<br/>
<br/>

## 빈 스코프

> **Scope : 범위**
> 
- 스프링이 지원하는 스코프
    1. 싱글톤 : 기본 스코프, 스프링 컨테이너가 시작되고 종료될 때까지 유지되다 → 가장 넓은 범위의 스코프
    2. 프로토타임 : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여, 더는 관리하지 않음 (떤저버린다) → 매우 짧은 범위의 스코프

<br/>
<br/>
<br/>
<br/>

## 프로토타입 스코프

> **싱글톤 빈 요청과 비교**
> 
- 싱글톤 빈 요청

![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/c7b9aef4-5a76-42e8-a346-a1f6bbc5a747)


1. 클라이언트가 싱글톤 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환한다.
3. 이후에 같은 요청이 와도 스프링 컨테이너는 같은 객체 인스턴스의 빈을 반환한다.

- **프로토 타입 빈요청 1**

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/a59ca4a4-b8ba-48e8-b11c-4ccc94fcd646)


1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 이 시점에 프로토 타입 빈을 이때 ! 생성하고, 필요한 의존관계를 주입한다.

![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/216e8d19-a228-4ef1-ac7d-a7b10832c992)


- 스프링 컨테이너는 항상 프로토타입 빈 요청에 대해서, 각기 다른 객체 인스턴스의 빈을 반환하고 앞으로 관리하지도 않는다.

스프링 컨테이너는 클라이언트의 빈을 반환하고 전혀 관리하지 않는다. **즉, 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다는 것이다.**  

→ 그러면 대체 누가 관리하는지? 

- 프로토타입 빈을 관리할 책임은 클라이언트에게 있다. → 따라서 @PreDestroy 같은 종료 메서드가 호출되지 않는다.

```java
public class PrototypeTest {

    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        System.out.println("find prototype1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find prototype2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);

        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
    }

    @Scope("prototype")
    static class PrototypeBean{
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }

    }
}
```

- 프로토타입 빈 생성

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/4b96994a-f6f2-4664-a4fe-d6407cfd7ca5)


1. 스프링 조회 시에 생성되고 초기화 메서드가 실행된다.
2. 서로 다른 참조값이 나타나는 것을 알 수 있다.
3. @PreDestroy가 작동하지 않는다.
    
    ```java
            prototypeBean1.destroy();
            prototypeBean2.destroy();
            // 직접 함수를 호출해서 종료해야함.
    ```
    

- 싱글톤 빈 생성

![Untitled 4](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/21cebc4e-eea5-4ee9-8581-3c5638695be7)


- 스프링 컨테이너에서 생성과 동시에 초기화를 하는 것을 볼 수 있다.
    - init이 먼저 나타나는 것을 확인.
- @PreDestroy가 작동되는 것을 확인함.

<br/>
<br/>

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

<br/>

![Untitled 5](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/6ca6b61c-e4e6-4e3b-a17b-09ab32f78c82)


- 가정 1
1. 클라이언트A와 B는 프로토타입 빈을 요청한다.
2. 프로토 타입 빈이기 때문에 각기 다른 빈을 생성하고 반환한다.
3. 반환된 객체는 count++를 한다.

<br/>

![Untitled 6](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/2082216c-0def-4f83-a85b-19cb7556dcd0)

- 가정 2
1. 클라이언트 A 는 그 이후 ClientBaen을 싱글톤으로 요청한다.
2. ClientBean의 의존관계는 자동 주입되어 초기화가 완료된다. 즉, 주입 시점에 빈을 생성하고 반환하게 된다.
3. ClientBean은 프로토타입 빈을 내부 필드에 보관한다(참조값을 보관만 했다고 가정 시, 프로토타입 빈의 count 필드 값은 0이다.)

<br/>

![Untitled 7](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/046de715-0cdd-428e-a89b-a04d2aa110ef)


- 가정 3
1. 클라이언트 A는 ‘clientBean.logic()’을 호출한다.
2. ClientBean은 프로토타입 빈의 ‘addCount()’를 호출해서 프로토타입 빈의 count를 증가시킬 수 있다. count값이 1이된다!

<br/>

![Untitled 8](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/66e9e256-0916-48f8-8274-ad06d91f297c)


- 가정 4
1. 클라이언트 B가 싱글톤인 clientBean을 요청하여 프로토타입 빈에서 addcount를 사용할 경우 count가 2로 늘어나게된다.
2. 이는 모든 클라이언트가 프로토타입 빈 또한 싱글톤처럼 자원을 공유해서 사용하고 있다는 이야기가 된다.

**우리의 의도가 만약 프로토타입을 요청할 때마다 항상 새롭게 만들고 싶은 것이다.** (이럴거면 싱글톤을 쓰지!! ) 

주입 시점에서 설정하는 것이아니라 → **사용할 때마다 새롭게 사용하고자 prototype을  사용하고 싶은 것.** 

- 이걸 해결 하는 가장 무식한 방법.

```java
    @Scope("singleton")
    @RequiredArgsConstructor
    static class ClientBean {
//        private final PrototypeBean prototypeBean; //생섬시점에 prototypeBean이 주입이 되어버리기 때문에 계속 같은 것을 사용한다.ㅇㅇㅇ
        private final ApplicationContext applicationContext;
        
        public int logic() {
            PrototypeBean prototypeBean = applicationContext.getBean(PrototypeBean.class);
            int count = prototypeBean.addCount();
            return count;
        }
    }

```

- ClientBean을 실행할 때마다 applicationContext를 이용하여 조회 시 새로운 객체를 생성해서 사용한다.

![Untitled 9](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/8bd5d6fb-3b8a-4e06-991b-b95e8c827716)


- 각각 새로 생성되고 count값이 독립적으로 반환됨.

- 이처럼 항상 새로운 프로토타입 빙니 생성되는 것을 볼 수 있다.
- 의존관계를 외부에서 주입(DI) 받는게 아니라 직접 필요한 의존관계를 찾는 것을 Dependency Lookup (DL) 의존관계 조회(탐색) 이라고 한다.
- 그런데 이렇게 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종석적인 코드가 되고 단위 테스트도 어려워진다!!

**딱 지정한 프로토타입 빈을 대신 찾아주는 DL 기능만 제공하는 무언가는 없을까???**

<br/>
<br/>
<br/>
<br/>

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용 시 Provider로 문제 해결

- 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공한다.

- ObjectProvider

```java
 @Scope("singleton")
    @RequiredArgsConstructor
    static class ClientBean {
//        private final PrototypeBean prototypeBean; //생섬시점에 prototypeBean이 주입이 되어버리기 때문에 계속 같은 것을 사용한다.ㅇㅇㅇ
        
        final private ObjectProvider<PrototypeBean> prototypeBeanProvider;

        public int logic() {
            PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
            int count = prototypeBean.addCount();
            return count;
        }
    }
    
    
   
```

- prototypeBeanProvider.getObject()를 통해 새로운 프로토타입 빈이 생성되는 것을 볼 수 있다.
- 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기 쉬워진다.

- Provider (javax 라이브러리)

```java
 @Scope("singleton")
    @RequiredArgsConstructor
    static class ClientBean {
//        private final PrototypeBean prototypeBean; //생섬시점에 prototypeBean이 주입이 되어버리기 때문에 계속 같은 것을 사용한다.ㅇㅇㅇ

        final private Provider<PrototypeBean> provider;

        public int logic() {
            PrototypeBean prototypeBean = provider.get();
            int count = prototypeBean.addCount();
            return count;
        }
    }
```

![Untitled 10](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/2f923db6-aa3c-4cae-b733-44e8d548ac21)


- 비슷하게 자바 표준으로 단위 테스트나 mock코드 만들기 용이하며
- 라이브러리만 가져오면 쉽게 사용이 가능하다.

- ObjectProvider, Provider은 프로토타입 빈에서 뿐만아니라 DL을 사용한다면 어디서든 사용 가능하다.

### 싱글톤 빈과 프로토 타입빈을 사용하는 일

- 프로토타입 빈을 따로 사용하는 경우는 거의없음..
- 예시
    - A가 B에 의존 , B가 A에 의존 할 경우 순환참조가 나타날 수 있음.
        - 이때 프로토타입 빈을 따로 등록하면 A는 B 당장 필요하지만 B는 A가 선택적으로 필요한 경우. (지연)
    
<br/>
<br/>
<br/>
<br/>


## 웹 스코프

- 특징
    - 웹 환경에서만 동작
    - 스프링이 해당 스코프의 종료 시점까지 관리한다. 따라서 종료 메서드가 호출된다.

- 종류
- request : HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.
- session : HTTP Session과 동일한 생명주기를 갖는 스코프
- application : 서블릿 컨텍스트(ServletContext)와 동일한 생명주기를 가지는 스코프
    - ServletContext란? → **웹 애플리케이션의 컨텍스트**. 웹 애플리케이션의 이름, 경로 및 초기화 파라미터를 포함한 웹 애플리케이션에 대한 정보를 포함
- websocket : 웹 소켓과 동일한 생명주기를 갖는 스코프

![Untitled 11](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/79c59063-edbb-44ef-8f74-d7264974513e)


**“HTTP에 맞춰서 각각 생성된다!”**

- 클라이언트 A ,B 요청에 맞추어서 각각 다른 스프링 빈이 생성 되어서 사용이된다.
- 클라이언트 A가 요청 → Controller가 조회되고 여기서 HTTP request에 따라 request scope에서 A전용 객체가 만들어진다.
- Controller 조회 → Service에서 조회 후 Logger 객체를 조회하면 HTTP request가 같으면 같은 객체 A를 바라보게 된다.

<br/>
<br/>
<br/>
<br/>

## request 스코프 예제 만들기

- 공통 포멧 : [UUID] [URL] {message}

```java
@Component
@Scope(value ="request")
public class MyLogger {
    private String uuid;

    @Setter
    private String requestURL;

    public void log(String message){
        System.out.println("[" + uuid + "]" + "[" + requestURL + "]" + message);
    }

    @PostConstruct
    public void init(){
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "]" + " request scope bean create: " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "]" + " request scope bean close: " + this);

    }

}
```

- @Scope(value ="request")를 이용하면, 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 끝나는 시점에서 소멸된다.
- @PostConstruct를 통해 생성과 동시에 UUID를 생성함
- @PreDestroy를 통해 빈이 종료되는 시점에 종료메세지를 남김.

- LogDemoController : HttpServletRequest를 이용하여 getRequestURI를 set함.

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logoDemo(HttpServletRequest request){
        String requestUrl = request.getRequestURI();
        myLogger.setRequestURL(requestUrl);
        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "ok";
    }

```

- LogDemoService

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final MyLogger myLogger;

    public void logic(String id){
        myLogger.log("service id = " + id);
    }
}
```

- 서버 실행 결과

> Error creating bean with name 'myLogger': Scope 'request' is not active for the current thread; consider defining a scoped proxy for this bean if you intend to refer to it from a singleton
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:373)…
> 

request scope는 실제 고객의 요청이 HTTP로 들어와야 사용이 가능하다. 

부분적으로 값을 제공받고 싶었을 경우에는 빈스코프 DL에서 썼던 ObjectProvider처럼 스코프 역시 ObjectProvider를 사용하자.

<br/>
<br/>
<br/>
<br/>

## 빈스코프와 Provider

- ObjectProvider를 사용

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logoDemo(HttpServletRequest request){
        MyLogger myLogger = myLoggerProvider.getObject();
        String requestUrl = request.getRequestURI();
        myLogger.setRequestURL(requestUrl);
        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "ok";
    }

}
```

```java

@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final ObjectProvider<MyLogger> myLoggers;

    public void logic(String id){
        MyLogger myLogger = myLoggers.getObject();
        myLogger.log("service id = " + id);
    }
}

<br/>
<br/>

```

![Untitled 12](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/3e480903-69a0-476e-8096-bbda54ac96d1)


![Untitled 13](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/0a5e2747-9963-4934-bdca-2410e0336cab)


- 스프링의 웹 스코프는 HTTP 요청 URL과 UUID를 매핑해서 이후 객체를 관리한다.
- 해당 빈을 생성하고 같은 빈을 destroy하는 것을 확인할 수 있다.
    - 개발자는 웹 스코프 빈을 HTTP 요청에 따라 나누어 관리하는 로직을 만들 필요 없이 스프링이 자동으로 도와준다.
- 만약 요청이 3개 있으면 Tread로 각각 스프링 빈을 할당해준다.

<br/>
<br/>
<br/>
<br/>

## 스코프와 프록시

- MyLogger에 프록시 모드를 추가.
    - 적용대상이 인터페이스가 아니면 proxyMode에 “TARGET_CLASS”를 사용한다.
    - 적용대상이 인터페이스가 아니면 proxyMode에 “INTERFACE”를 사용한다.
 
```java
@Component
@Scope(value ="request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
```

- 서버 실행 시 HTTP 요청이 없는 상태에서 웹스코프를 사용하더라도 오류가 생성되지 않는다.
    - MyLogger에 가짜 프록시 클래스를 만들어두고  **HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입할 수 있다.**

<br/>
<br/>

- 프록시를 사용하면서 CGLIB라는 가짜 클래스를 스프링 프록시 객체를 만들어서 의존관계를 주입한다.

```java
myLogger = class hello.core.common.MyLogger$$SpringCGLIB$$0
```

> **이 가짜 프록시 객체는 요청이 들어오면 그때 내부에서 진짜 빈을 위임하는 로직이 들어가있다!**
> 

- 가짜 프록시 객체는 원본 클래스를 상속받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사실 원본인지 아닌지 모르게, 동일하게 사용할 수 있다 → **“다형성”**
    - 단지 어노테이션 설정 변경만으로도 원본 객체를 프록시 객체로 대체할 수 있다. → **다형성과 DI의 힘!**
- 가짜 프록시 객체는 request scope와는 전혀 관련없이 싱글톤 처럼 동작할 뿐이다.

<br/>
<br/>

**“정리”**

- 프록시 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯 request scope를 사용할 수 있다.
- 정말 중요한 부분은 필요한 시점까지 해당 request scope를 지연 처리할 수 있다는 것이다.
    
<br/>
<br/>

**“주의점”**

- 싱글톤으로 동작하는 것 같지만 독립적으로 동작하기 때문에 주의가 필요하다.
- 무분별하게 사용하면 유지보수가 어렵다.
