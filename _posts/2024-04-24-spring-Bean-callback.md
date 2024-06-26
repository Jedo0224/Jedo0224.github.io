---
title:  "빈 생명주기 콜백"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Spring
tags:
  - [Spring, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-04-23
last_modified_at: 2024-04-24
---

## 강의 링크
[스프링 핵심 원리 - 기본편 강의 - 인프런](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)


<br/>

<br/>

## 빈 생명주기 콜백 시작

- 빈, 객체의 초기화 및 종료 작업을 지원 예시
    1. 먼저 데이터베이스와 서버를 연결시켜 놓는 커넥션 풀을 설정하는 경우 
    2. 네트워크를 가지고 서버를 연결하기위해 소캣을 미리 열어놓는 것들에 경우, 
    
    → 애플리케이션 시작 시점에 작업되며 이것들을 안전하게 종료하는 것까지 스프링이 관리해준다.
    

- **서버가 시작될 때 바로 외부 네트워크가 연결되고 종료될때 미리 종료되는 외부 네트워크가 있다고 가정.**

```java
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출= " + url);
        connect();
        call("초기화 연결 메시지");
    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작 시 호출
    public void connect(){
        System.out.println("connect: = " + url);
    }

    public void call (String message){
        System.out.println("call = " + url +" message = " + message);
    }

    //서비스 종료 시 호출
    public void disconnect(){
        System.out.println("close");
    }
}

```

```java
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest(){
        //ConfigurableApplicationContext가 AnnotationConfigApplicationContext의 부모
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close(); //AnnotationConfigApplicationContext은 close를 제공해주지 않음.
    }

    @Configuration
    static class LifeCycleConfig{
        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }
}
```
![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/d9d18920-d4bb-48d6-970c-b22efb71e938)

<br/>
<br/>


위와 같은 결과가 나타나는 이유.

- 생성자를 이용해 객체를 생성할 경우, 객채 내에 아무런 값도 주입되지 않았기 때문.
    - 따라서 외부에서 setUrl을 해주어야만함. → 하지만 생성자에 바로 값을 주입하는 방법도 존재함.
- 스프링빈은 **“객체 생성” → 의존관계 주입”**의 라이프사이클을 가지고 있는 것이다.
    - 생성자 주입은 예외. 나머지는 위와 같다.

초기화는 객체를 생성하는 작업을 말하는게 아니다.

- 객체 안에 필요한 값들이 **주입된 후에 연결을 맺는 행위**이다. → 그러면 개발자는 **의존관계 주입이 전부 완료되었는지 어떻게 알 수 있을까?**

<br/>
<br/>

### 그러면 그냥 생성자 주입만 쓰면 되지 않을까?

- 객체의 생성과 초기화를 분리하는것이 좋다.
    - 단일 책임 원칙 하에서 객체 생성은 객체를 생성하는 것에만 집중하는 것이 좋다.
    - 외부 네트워크와의 연결은 무거운 동작에 해당한다.(서버에 영향을 크게 줄 수 있는 동작)
    - 무겁게 초기화 작업을 함께하는 것보다, 생성과 초기화를 명확하게 나누는 것이 유지 보수 측면에서 좋다.
    - 또는 생성만 하고 기다리다가 어떠한 최초의 액션이 주어지면 그때 초기화를 하는 방법을 사용해서 네트워크 자원 효율을 얻을 수도 있다.


<br/>
<br/>


### 스프링의 초기화 및 종료 작업 지원

“스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공한다.”

- 스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 준다.

- 스프링 빈의 이벤트 라이브사이클
    - 스프링 컨테이너 생성 → 스프링 빈 생성 → **의존관계 주입 → 초기화 콜백** → 사용 → **소멸전 콜백** → 스프링 종료
    
- 빈 생명 주기 콜백 종류
    - 인터페이스 (InitializingBean, DisposableBean)
    - 설정 정보에 초기화 메서드, 종료 메서드 지정
    - @PostConstruct, @PreDestrory 어노테이션 지원

<br/>
<br/>
<br/>
<br/>

## 인터페이스 InitializingBean, DisposableBean

```java
public class NetworkClient implements InitializingBean , DisposableBean {

//...
	
	
	//객체 생성 후 어떤 행동을 취할지 콜백
  @Override
    public void afterPropertiesSet() throws Exception {
		    System.out.println("NetworkClient.afterPropertiesSet");
        connect();
        call("초기화 연결 메시지");
    }

	//객체 종료 후 어떤 행동을 취할지 콜백
    @Override
    public void destroy() throws Exception {
		    System.out.println("NetworkClient.destroy");
        disconnect();
    }

}
```

- 결과
    
    
    ![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/f7dbf95b-e52a-4f34-98e2-da712307877d)


    - 생성자를 이용해 객체 생성 → 의존성 주입 → 초기화 (초기화 콜백) → 종료
    

> 인터페이스 (InitializingBean, DisposableBean)의 단점
> 
1. 스프링 전용 인터페이스 이기 떄문에 해당 코드가 스프링 전용 인터페이스에 의존한다.
2. 초기화, 소멸 메서드의 이름을 변경할 수 없다.
3. 내가 고칠 수 없는 외부 라이브러리에 적용이 불가능함.

→ 초창기에 나온 방법 **거의 사용하지 않는다.**

<br/>
<br/>
<br/>
<br/>

## 설정 정보에 초기화 메서드, 종료 메서드 지정

```java
public class NetworkClient {

//...
	
	
	//객체 생성 후 어떤 행동을 취할지 콜백
  public void init() throws Exception {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 연결 메시지");
    }

    public void close() throws Exception {
        System.out.println("NetworkClient.close");
        disconnect();
    }

}
```

- `BeanLifeCycleTest`

```java
  @Configuration
    static class LifeCycleConfig{
        @Bean(initMethod = "init", destroyMethod = "close")
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }
```

 @Bean(initMethod = "init", destroyMethod = "close")

> **설정 정보 사용 장점**
> 
1. 메서드 이름을 자유롭게 지정 가능.
2. 스프링 빈이 스프링 코드에 의존하지 않음.
3. 코드가 아니라 설정 정보를 사용하기 때문에 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있음.

- 종료 메서드 추론
    - @Bean의 destroyMethod는 기본값이 ‘(inferred)’ (추론)으로 등록되어 있다.
    - 이 추론 기능은 ‘close’와 ‘shutdown’이라는 메서드를 자동으로 호출한다.
    - 따라서 **스프링 빈으로 등록하면 종료 메서드를 따로 적어주지 않아도 된다.**
    - 이 기능을 사용하기 싫으면 destroyMethod = ""를 사용하자.
    
    AutoClose?
    
<br/>
<br/>
<br/>
<br/>

## @PostConstruct, @PreDestrory 어노테이션 지원

PostConstruct : 미리 만들다

PreDestrory : 미리 부순

```java
public class NetworkClient {

//...
	
	
	 @PostConstruct
    public void init(){
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 연결 메시지");
    }

    @PreDestroy
    public void close(){
        System.out.println("NetworkClient.close");
        disconnect();
    }
}
```

> **@PostConstruct, @PreDestrory**
> 
1. 최신 스프링에서 **가장 권장하는 방법**이다.
2. 어노테이션만 붙이면 되기에 매우 편리하다.
3. javax를 사용하기 떄문에 스프링 종속이 아닌 자바 표준이기 때문에 다른 컨테이너에서도 동작한다.
4. **하지만 외부 라이브러리에는 적용하지 못한다**. 이 경우 @Bean의 설정 정보에 초기화 메서드, 종료 메서드 지정를 사용하자.
