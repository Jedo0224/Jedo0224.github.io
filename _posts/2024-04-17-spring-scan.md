---
title:  "컴포넌트 스캔"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Spring
tags:
  - [Spring, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-04-16
last_modified_at: 2024-04-17
---

## 강의 링크
[스프링 핵심 원리 - 기본편 강의 - 인프런](https://www.inflearn.com/course/스프링-핵심-원리-기본편/dashboard)

## 컴포넌트 스캔과 의존관계 자동 주입 시작하기

- 스프링 빈을 일일이 빈에 등록하기 귀찮음.
- 설정 정보가 커지면 누락하는 문제도 발생함.

→ 따라서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 “**컴포넌트 스캔**”이라는 기능을 제공하고, 해당 등록된 bean을 자동으로 의존관계를 주입시키는 **@Autowired**라는 기능을 제공한다.

@Compoent를 사용하게 되면 @Autowired를 사용하게 된다.

```java
@Component
public class MemberServiceImpl implements MemberService{
    private final MemberRepository memberRepository;

    @Autowired // ac.getBean(MemberRepository.class)
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```

- @Autowired는 ac.getBean(MemberRepository.class)와 역할이 비슷하며 자동으로 의존관계를 주입한다.

```java
public class AutoConfigTest {

    @Test
    void basicScan(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
        MemberService memberService = ac.getBean(MemberService.class);

        Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
    }
}
```

![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/0e21ce56-3eca-4213-ae08-db6a480d4733)



- 테스트 로그를 통해서 싱글톤 bean이 생성된것과 Autowired로 자동으로 의존관계가 주입된 것을 볼 수 있다.

<br/>
<br/>

### @CompoentScan

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/a9aef594-7da4-4d99-989b-68446e1ccb0e)


- @Compoent가 붙은 모든 클래스를 스프링 bean으로 등록한다.
- bean 등록에 경우 기본 이름을 클래스 이름으로 사용하되 맨 앞글자만 소문자를 사용한다.

<br/>
<br/>

### @Autowired 의존관계 자동주입

- 생성자에 @Autowired를 지정하면 자동으로 해당 스프링 빈을 찾아서 주입한다.
    
    ![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/e0e04223-7cae-4470-b12e-c9f781209121)

    
    - 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다. == getBean(MemberRepository.class)
    - 생성자에 파라미터가 많아도 타입이 같은 빈을 찾아서 주입한다.

<br/>
<br/>
<br/>
<br/>

## 탐색 위치와 기본 스캔 대상

```java

@Configuration
@ComponentScan (
        basePackages = "hello.core.member",
        basePackageClasses = AutoAppConfig.class,
        excludeFilters =  @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
) //자동으로 스프링 빈에 등록한다, 하지만 AppConfig, TestConfig 등 우리가 수동으로 스프링 빈에 등록한 @Configuration class들을 제외한다 제외한다.

// @Bean으로 등록하지 않아도 된다.
public class AutoAppConfig {

}

```

- 전체 JAVA 코드에 대한 bean을 등록하는 overhead를 줄이기 위해 스캔 범위를 설정할 수 있다.(basePackages , basePackageClasses)
- 범위가 설정되지 않은 값은 해당 클래스가 위치한 패키지를 최상위로 두어 스캔을 진행한다.

### Trouble shooting

    - **org.springframework.beans.factory.UnsatisfiedDependencyException**
        
        → 내가 AutoConfig를 잘못된 패키지에 설정했기 때문에 Bean이 생성되지않았고 해당 Bean이 생성되지 않았기에 의존성이 불완전한 UnsatisfiedDependencyException가 발생했음.
        
    - `@SpringBootApplication`를 사용하는 클래스가 루트 패키지에 존재하는 이유

### @Component를 사용하는 다른 어노테이션

    - @Controller : 스프링 MVC 컨트롤러에서 사용
    - @Service : 스프링 비즈니스 로직에서 사용 (핵심 비즈니스 로직이 여기에 있다는 것을 개발자가 인식)
    - @Repository : 스프링 데이터 접근 계층에서 사용 (데이터 계층의 예외를 스프링 예외로 변형)
    - @Configuration : 스프링 설정 정보에서 사용 (설정 정보 및 싱글톤을 유지하기 위한 추가 처리)

<br/>
<br/>
<br/>
<br/>

## 중복 등록과 충돌

1. 자동 bean 등록 vs 자동 bean 등록
    
    → ConflictingBeanDefinitionException, BeanDefinitionStoreException 발생하여 conflict를 방지한다.
    
2. 수동 bean 등록 vs 자동 bean 등록

```java
public class AutoAppConfig {
		// 수동 bean을 등록하여 중복을 만든다.
    @Bean(name = "memoryMemberRepository")
    MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

}
```

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/9e2c6dee-b5f6-4ca6-b1ed-7563217e31d9)


→ Overriding bean definition for bean 'memoryMemberRepository' with a different definition: replacing 로그 출력.

- 최근 스프링 부트에서 수동 baen과 자동 bean이 충돌하면 자동으로 예외를 발생시키도록 변경되었다.
    - setting spring.main.allow-bean-definition-overriding=true를 따로 설정해주어야 충돌 시 overriding을 허용할 수 있음. (default : false)
