---
title:  "스프링 컨테이너와 빈"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Spring
tags:
  - [Spring, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-04-04
last_modified_at: 2024-04-05
---


## 강의 링크
[스프링 핵심 원리 - 기본편 강의 - 인프런](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)


## 스프링 빈 조회 - 기본
```jsx

AnnotationConfigApplicationContext  ac = new  AnnotationConfigApplicationContext(AppConfig.class);

@Test

@DisplayName("모든 빈 출력하기")

void  findAllBean() {

String[] beanDefinitionNames = ac.getBeanDefinitionNames();

for (String  beanDefinitionName : beanDefinitionNames){

Object  bean = ac.getBean(beanDefinitionName);

System.out.println("name = " + beanDefinitionName + "object = " + bean);

}

}

  

//Role ROLE_APPLICATION: 직접 등록한 애플리케이션 빈

//Role ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈

  

@Test

@DisplayName("애플리케이션 빈 출력하기")

void  findApplicationBean() {

String[] beanDefinitionNames = ac.getBeanDefinitionNames();

for (String  beanDefinitionName : beanDefinitionNames){

BeanDefinition  beanDefinition = ac.getBeanDefinition( beanDefinitionName );

  

if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){

Object  bean = ac.getBean(beanDefinitionName);

System.out.println("name = " + beanDefinitionName + "object = " + bean);

}

}

```

  

- BeanDefinition에 ROLE_APPLICATION / ROLE_INFRASTRUCTURE의 상수를 설정을 통해 “직접 등록한 빈” / “스프링이 내부에서 사용하는 빈”을 확인할 수 있음.

- 기본값은 스프링 내부 + 직접 만든 빈이 출력됨.

<br/>  
<br/>

### 성공 테스트

  

```jsx

class  ApplicationContextBasicFindTest {

  

AnnotationConfigApplicationContext  ac = new  AnnotationConfigApplicationContext(AppConfig.class);

  

// @Test

// @DisplayName("빈 이름으로 조회")

// void findBeanByName() {

// MemberService memberService = ac.getBean("memberService", MemberService.class);

// System.out.println("memberService = " + memberService);

// System.out.println("memberService.getClass() = " + memberService.getClass());

// }

  

@Test

@DisplayName("빈 이름으로 조회")

void  findBeanByName() {

MemberService  memberService = ac.getBean("memberService", MemberService.class);

assertThat(memberService).isInstanceOf(MemberServiceImpl.class);

//memberService가 MemberServiceImpl.class의 instance면 성공

}

  

@Test

@DisplayName("이름 없이 타입으로만 조회")

void  findBeanByType() {

MemberService  memberService = ac.getBean(MemberService.class);

assertThat(memberService).isInstanceOf(MemberServiceImpl.class);

}

}

```

  

```jsx

@Test

@DisplayName("구체 타입으로 조회")

void  findBeanByName2() {

MemberService  memberService = ac.getBean("memberService", MemberServiceImpl.class);

assertThat(memberService).isInstanceOf(MemberServiceImpl.class);

}

```

  

- 이와 같이 구체 타입으로 빈이 조회가 가능하다.

- 항상 역할과 구현을 구분해야한다. 역할에 의존해야기 때문에 구체에 의존한 코드는 좋은 코드가 아니다.

![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/772dcda4-31aa-4bfb-a04b-156f34eaed45)


<br/>  
<br/>

### 실패 테스트

  

```jsx

@Test

@DisplayName("빈 이름으로 조회 X")

void  findBeanByNameX() {

ac.getBean("XXXX", MemberService.class);

}

//org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'XXXX' available

```
<br/>  
<br/>
  

## 동일한 타입이 둘 이상일 경우

  

```jsx

  

AnnotationConfigApplicationContext  ac = new  AnnotationConfigApplicationContext(SameBeanConfig.class);

  

@Test

@DisplayName("타입으로 조회 시 같은 타입이 둘 이상 있으면 , 중복 오류가 발생한다.")
void  findBeanByTypeDuplicate(){
// MemberRepository bean = ac.getBean(MemberRepository.class);
assertThrows(NoUniqueBeanDefinitionException.class,() -> ac.getBean(MemberRepository.class));

}

  

@Test
@DisplayName("타입으로 조회 시 같은 타입이 둘 이상 있으면, Bean 이름을 지정하면 된다.")
void  findBeanByName() {
MemberRepository  memberRepository = ac.getBean("memberRepository1",MemberRepository.class);
Assertions.assertThat(memberRepository).isInstanceOf(MemberRepository.class);
}

  

@Test

@DisplayName("특정 타입을 모두 조회하기")

void  findAllBeanByType() {
Map<String, MemberRepository> beansOfType =ac.getBeansOfType(MemberRepository.class);

for (String  key : beansOfType.keySet()) {
System.out.println("key = " + key + " value = " + beansOfType.get(key));
}

System.out.println("baensOfType = " + beansOfType);
Assertions.assertThat(beansOfType.size()).isEqualTo(2);

}

  

@Configuration

static  class  SameBeanConfig {

@Bean

public  MemberRepository  memberRepository1() {
return  new  MemoryMemberRepository();
}

  

@Bean

public  MemberRepository  memberRepository2() {
return  new  MemoryMemberRepository();
}

}

  

```

  

- 자동으로 의존관계가 주입 될 경우에 특정 타입을 모두 조회하는게 자동으로 이루어진다.

<br/>  
<br/>

## 상속 관계

  

- 부모 타입으로 조회하면 , 자식 타입도 함께 조회한다.

- 모든 자바 객체의 최고 부모인 Object 타입으로 조회하면 , 모든 스프링 빈을 조회한다.

  

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/eb7721e7-190a-42db-b680-93563df09576)


  

- 실제 개발 시에는 Context Bean을 조회할 일은 거의 없다

  

→ 기본적인 의존주입 과정을 이해해야하며 , 가끔 순수한 Java 어플리케이션에서 스프링 컨테이너를 생성해서 사용해야할 경우가 있을 수 있다.

<br/>  
<br/>
  

## BeanFactory와 ApplicationContext

  

![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/788dca04-1e5c-4fc6-af16-1da041c1b342)


  

- ApplicationContext는 BeanFactory에 부가 기능을 추가한 것.

<br/>  
<br/>

### BeanFactory

  

- 스프링 컨테이너의 최상위 인터페이스

- 스프링 빈을 관리하고 조회하는 역할을 담당한다.

- ‘getBean()’을 제공한다.

  <br/>  
<br/>

### ApplicationContext

  

- BeanFactory 기능을 모두 상속 받아서 제공한다.

- Bean을 관리하고 검색하는 기능을 BeanFactory가 제공해준다.

- 애플리케이션을 개발할 때는 Bean을 관리하고 조회하는 기능과 함께 **수많은 부가 기능**이 필요하다

  <br/>  
<br/>

### ApplicationContext가 제공하는 부가 기능

  

```jsx

public  interface  ApplicationContext  extends  EnvironmentCapable, ListableBeanFactory,

HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver

```

  

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/2235979d-5d64-49d3-9d2d-47956b01c111)


  

-  **“메세지 소스를 활용한 국제화 기능”**

  - 한국에서 들어오면 한국어로, 영어권에서는 영어로 출력하는 웹사이트 기능

  - 파일을 여러개 분리해놓고 해당 인수에 대한 메세지소스 형태의 파일을 제공

-  **“환경변수를 구분하는 기능”**

  - 실제 개발할 경우… 크게 3가지 환경

  - 로컬 , 개발(테스트 서버에 띄워두고 개발) , 운영(환경) 등을 구분해서 처리

-  **“애플리케이션 이벤트”**

  - 이벤트 발행 및 구독 모델을 편리하게 지원한다.

  -  **“편리한 리소스 조회”**

  - 파일, 클래스 path, 외부 등에서 리소스를 편리하게 조회

<br/>  
<br/>

## 스프링 빈 설정 메타 정보 - BeanDefinition

  

- BeanDefinition이라는 추상화를 통해 다양한 설정 형식을 지원한다(ex. xml, java …)

![Untitled 4](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/5e3da95e-a8a5-415a-bcf1-b5f6c36347dd)


  - 역할과 구현을 개념적으로 나눈 것

  - XMl을 읽어서 또는 Java를 읽어서 → BeanDefinition

  - 스프링 컨테이너는 자바 코드인지 , XML인지 몰라도 된다. 오직 BeanDefintion만 알면된다.

  - ‘BeanDefintion’을 빈 설정 정보라고 한다.

![Untitled 5](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/51717d97-e4d6-406a-aaa3-80b8de2c6075)

  - Appconfig.class / .xml 를 읽어내서 설정 정보를 BeanDefinition에 메타정보를 저장한다.

```java

public  class  BeanDefinitionTest {

AnnotationConfigApplicationContext  ac = new  AnnotationConfigApplicationContext(AppConfig.class);

@Test

@DisplayName("빈 설정 메타정보 확인")

void  findApplicationBean() {

String[] beanDefinitionNames = ac.getBeanDefinitionNames();

for (String  beanDefinitionName  : beanDefinitionNames) {

BeanDefinition  beanDefinition = ac.getBeanDefinition(beanDefinitionName);

if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){

System.out.println("beanDefinitionName = " + beanDefinitionName + " beanDefinition = " + beanDefinition);

}

}

}

```

-  **“BeanDefinition 정보”**

- Scope : 싱글톤(기본값)

  - 빈이 어떻게 생성되고, 애플리케이션 내에서 공유되는지를 결정

  -  **Singleton**: 기본값으로, 스프링 컨테이너당 빈 인스턴스가 하나만 생성된다. 이는 같은 애플리케이션 컨텍스트 내에서 요청되는 모든 참조에 대해 동일한 인스턴스가 반환된다.

  -  **Prototype**: 요청할 때마다 새로운 빈 인스턴스가 생성된다. 이 스코프는 각각의 빈 요청에 대해 고유한 인스턴스가 필요한 경우에 사용한다.

- lazyinit : 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연 처리 하는지 여부

  - 이 설정은 애플리케이션의 시작 시간을 단축시킬 수 있다. 특히, 애플리케이션에 많은 수의 빈이 있거나 빈의 초기화 과정이 복잡하고 시간이 많이 걸릴 때 유용하다.

- initMethodName : 빈을 생성하고 , 의존관계를 적용한 뒤 호출되는 초기화 메서드 명

- DestroyMethodName : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명

- Constructor argument, Properties : 의존관계 주입에서 사용된다.

<br/>
<br/>

### 정리

- BeanDefinition을 직접 생성해서 스프링 컨테이너에 등록 및 정의할 수 있다. (실무에서는 거의 하지 않는다.)

- 스프링이 다양한 형태의 설정 정보를 BeanDefinition으로 추상화해서 사용하는 것 정도이해하자.
