---
title:  "연관관계의 이해"
excerpt: "스프링 부트 핵심 원리 - 기본편"

categories:
  - Jpa
tags:
  - [Spring, Java, OOP,Jpa]

toc: true
toc_sticky: true
 
date: 2024-06-03
last_modified_at: 2024-06-04
---


## 목표

- 객체와 테이블의 연관관계를 이해한다.
- 객체의 참조와 테이블의 외래키를 매핑한다.
- 용어 이해
    - 방향 : 단방향, 양방향
    - 다중성 : 다대일(N:1) , 일대다(1:N) , 일대일(1:1), 다대다(N:M) 이해
    - 연관관계의 주인 : 객체 양방향 연관관계 관리가 필요.

<br/>
<br/>

## 연관관계가 필요한 이유

‘**객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다**’ - 조영호(객체지향의 사실과 오해) 

- 객체 하나, 테이블 하나로는 시스템을 만들 수 없다, 연관관계를 활용해야한다.

<br/>
<br/>

## 단방향 연관관계

### 객체를 테이블에 맞춘 모델링

- Member
    
    ```python
    @Entity
    @Getter
    public class Member {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name="MEMBER_ID")
        private Long id;
        @Column(name = "USERNAME")
        private String username;
        @Column(name = "TEAM_ID")
        private String teamId;
    
    }
    ```
    

- Team
    
    ```python
    @Entity
    public class Team {
    
        @Id
        @GeneratedValue
        @Column(name = "TEAM_ID")
        private Long id;
    
        @Column(name = "TEAM_NAME")
        private String name;
    }
    ```
    

- JpaMain - 저장
    
    ```python
    tx.begin();
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setUsername("member1");
    member.setTeamId(team.getId());
    em.persist(member);
    
    tx.commit();
    ```
    
    - 결과
    
    ![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/c3c43d73-5db2-47ac-9190-692f4a2ba35e)


    - Member_ID가 2인 이유는 h2가 자동적으로 sequence 매핑을 하기 때문이다.
    - select * from team t join member m on m.team_id = t.team_id;
        
        ![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/1bba7e52-af70-4bd5-89fb-ca0c902ac7ef)

        

- JpaMain - 조회 , 해당 멤버의 팀이 어디인지
    
    ```java
    Member findMember = em.find(Member.class, 2L);
    Team findTeam = em.find(Team.class,findMember.getTeamId());
    
    System.out.println("2's Team: " + findTeam.getName());
    ```
    
    멤버를 조회 → 그 멤버가 속한 팀을 조회
    
    객체를 테이블에 맞추어 데이터 중심으로 모델링하면 협력 관계를 만들 수 없다. (협력관계??)
    
    - 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.

<br/>
<br/>


## 객체 지향 모델링 (객체 지향 연관관계 사용)

- 테이블의 참조값을 이용한다.

![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/fe54fbc0-5eaf-4426-b471-14122a9b7110)

- member
    
    ```java
    public class Member {
        @Id
        @GeneratedValue()
        @Column(name="MEMBER_ID")
        private Long id;
        @Column(name = "USERNAME")
        private String username;
    //    @Column(name = "TEAM_ID")
    //    private Long teamId;
    
        @ManyToOne // Member 입장에서는 many , Team 입장에서는 one
        @JoinColumn(name = "TEAM_ID") 
        private Team team;
    }
    ```
    
    - @ManyToOne 관계에 대한 @JoinColumn을 지정해주면 매핑이 완료된다.
    
- JpaMain
    
    ```java
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setUsername("member1");
    member.setTeam(team);
    em.persist(member);
    
    em.flush()
    em.clear()
    
    Member findMember = em.find(Member.class, 2L);
    //            Team findTeam = em.find(Team.class,findMember.getTeamId());
    Team findTeam = findMember.getTeam();
    System.out.println("2's Team: " + findTeam.getName());
    ```
    
    - 테이블 매핑은 두번의 쿼리가 나간다.
        1. select * from member m where [m.id](http://m.id) = 2
        2. 위 쿼리의 결과에서 team_id를 조회한다.
        3. select * from team t where t[.id](http://m.id) =1 
        
    - 객체지향 연관관계 매핑은 join문 쿼리가 생성된다.
        1. select * from member m join team t on m.team_id = [t.id](http://t.id) where [m.id](http://m.id) = 2
    
<br/>
<br/>

## 양방향 연관관계와 연관관계의 주인

### 양방향 매핑

<그림 a : 양방향 객체 연관관계>

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/71e13b24-12ef-412e-bc56-c2e7e616f2d5)

- <그림 2> 처럼 양방향 객체 연관관계를 활용하더라도 DB의 테이블 연관관계는 바뀌지 않는다.
- 테이블 연관관계는 TEAM의 입장에서도 TEAM_ID(PK)를 이용하여 JOIN하면 “해당 팀에 속한 MEMBER를 조회할 수 있다.
- 하지만 단방향 객체 연관관계는 TEAM 엔티티에 MEMBER 속성이 없기 때문에 위와 같은 조회는 불가능하다 → 양방향 연관관계를 활용하자.

- Member

```java
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    @Column(name = "TEAM_NAME")
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

- @OneToMany(mappedBy = "team") : 나의 PK값을 연결된 테이블에 어떤 이름의 FK로 사용되고 있는지.
- **mappedBy : 양방향 매핑 (사실 양방향 X ,  단반향 2개)**
    
    객체 단방향 연관관계 2개를 합친 것이기 때문에 두개의 참조값이 필요하다.
    
    단방향 + 단방향을 하기위해 mappedBy를 지정하는 것이다. (추가적으로 참조값을 지정하기 위해)
    
<br/>
<br/>

## 양방향 연관관계의 딜레마

<그림 b : 참조값 두 개중 외래 키를 관리해야한다>

![Untitled 4](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/bb9edab0-38cd-4e71-af4e-69a4a5760cbb)

1. Member의 Team이 바뀌었을때 DB의 FK가 업데이트 되어야할까?
2. Team의 List member가 바뀌었을때 DB의 FK가 업데이트 되어야할까?

→ **주인을 정해야한다!** 

<br/>
<br/>

## 연관관계의 주인(Owner)

### 양방향 매핑 규칙

- 객체의 두 관계중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래 키를 관리(등록, 수정)
- 주인이 아닌쪽은 읽기만 가능하다.
- 주인은 mappedBy 속성을 사용하지 않는다 (그 엔티티에 mappedBy가 있으면 해당 관계에서 그 엔티티는 주인이 아니다)
- 주인이 아니면 mappedBy 속성으로 주인 지정

<br/>

### 누구를 주인으로 설정하는가?

- 실제 외래 키가 있는 곳이 주인으로 해라.
    - <그림 c : Member의 외래키 team이 연관관게의 주인으로 설정>
    
    ![Untitled 5](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/6ca7bbf1-78e9-4ac6-9e5a-f804e2ec8b86)


- 만약 Teaml의 List memers를 주인으로 설정한다면..
    - Team 엔티티에 값을 변경했는데 Member 테이블이 변경되는 것. → 객체지향적이지않고 성능 이슈가 존재한다.

- N(다 , toOne)에 해당하는 부분이 연관관계의 주인이된다,

<br/>
<br/>

## 양방향 연관관계 - 주의점, 정리

### 양방향 매핑시 가장 많이 하는 실수

- <코드 a : : 연관관계 주인이 아닌 속성에 값을 업데이트 했을 경우>
    - 역방향(주인이 아닌 방향)에만 값을 설정.

```java
 Member member = new Member();
      member.setUsername("member1");
      em.persist(member);

      Team team = new Team();
      team.setName("TeamA");
      team.getMembers().add(member);
      em.persist(team);

      em.flush();
      em.clear();
```

- <그림 d : 연관관계 주인이 아닌 속성에 값을 업데이트 했을 경우>
    - mappedby로 읽기 전용으로 사용되기 때문에 값이 생성되지 않는다.

![Untitled 6](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/0060ae46-ad55-4845-ae1f-06afc1d3c98d)

- <코드 b : 연관관계 주인 속성에 값을 업데이트 했을 경우>
    
    ```java
       Team team = new Team();
            team.setName("TeamA");
    //            team.getMembers().add(member);
            em.persist(team);
    
            Member member = new Member();
            member.setUsername("member1");
            member.setTeam(team);
            em.persist(member);
    ```
    

- <그림 d : 연관관계 주인인 속성에 값을 업데이트 했을 경우 결과>

![Untitled 7](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/a8c1895c-bde7-4c1a-8eee-9e64b362b592)

<br/>
<br/>

## 객체지향적으로 주종 관계 둘다 값을 넣어주는게 좋다.

<코드 b : flush를 사용하지 않을 경우 업데이트 되는 값을 확인할 수 없게됨>

```java
em.flush();
em.clear();

Member findMember = em.find(Member.class, 2L);
//Team findTeam = em.find(Team.class,findMember.getTeamId());
List<Member> findTeamMembers = findMember.getTeam().getMembers();

for (Member memberName :  findTeamMembers){
    System.out.println("2's Team: " + memberName.getUsername());
};

```

- em.flush(), em.clear()를 통해 DB에 해당 Member를 업데이트하는 쿼리를 보낸 후에 TEAM에서 Member를 조회할 수 있게된다

→ 쿼리를 보내지 않으면 1차 캐시에는 TEAM 테이블에 Members를 속성이 업데이트 되지않았기 때문에 조회할 수 없다.

→ 쿼리를 보내고 조회해야지 DB값을 Entity로 매핑할때 List<Member> members를 업데이트하고 조회할 수 있다.

1차 캐시(Java Collection)에도 값을 업데이트 해줘야 지연로딩의 영향을 받지않고 조회 및  테스트 케이스에 값을 활용할 수 있다.

- 연관관계 편의 메소드를 생성하자
    
    상황에 맞게 1 또는 N에 생성하면 된다.
    
    - <코드 d  : member에 Team을 새롭게 저장할 경우 , team의 members를 업데이트
    
    ```java
    public void ChangeTeam(Team team){
            this.team = team;
            team.getMembers().add(this);
        }
    ```
    

- 양방향 매핑시에 무한 루프를 조심하자
    - toString(), lombok, JSON 생성 라이브러리를 이용할 경우 생길 수 있음.
        - lombok에서는 웬만하면 tostring을 쓰지말자.
        - 컨트롤러에서 entity를 절대 반환하지말자. (DTO로 변환해서 반환하라!)

<br/>
<br/>

## 양방향 매핑 정리

- 단방향 매핑만으로도 이미 연관관계 매핑은 완료시켜야함.
    - 양방향은 필요할 때 추가해도 됨( 테이블에 영향을 주지 않기 때문 )
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것일 뿐
- JPQL에서 역방향으로 탐색할 일이 많음
