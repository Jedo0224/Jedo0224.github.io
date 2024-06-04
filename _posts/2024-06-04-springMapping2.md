---
title:  "다양한 연관관계 매핑"
excerpt: "자바 ORM 표준 JPA 프로그래밍 - 기본편"

categories:
  - Jpa
tags:
  - [Jpa, Java, OOP]

toc: true
toc_sticky: true
 
date: 2024-06-03
last_modified_at: 2024-06-04
---




## 연관관계 매핑시 고려사항 3가지

1. 다중성
2. 단방향 , 양방향 
3. 연관관계의 주인

<br/>

## 다중성

다대일 : @ManyToOne

일대다 : @OneToMany

일대일 : @OneToOne

다대다 : @ManyToMany

<br/>
<br/>

## 단방향, 양방향

- 테이블

외래키 하나로 양쪽으로 조인할 수 있다. 따라서 사실 테이블은 방향이라는 개념이 없다고 볼 수 있다.

- 객체

참조용 필드가 있는 쪽으로만 참조가 가능하다. 따라서 방향이 존재하며, 한쪽만 참조하면 단방향 , 양쪽이 서로 참조하면 양방향이다.(서로가 단방향 관계를 맺고있다.)

<br/>
<br/>

## 연관관계의 주인

테이블은 외래 키 하나로 두 테이블이 연관관계를 맺는다. 하지만 객체의 양방향 관계는 서로가 단방향 관계를 맺고 있기 때문에 (A→B , B→A) 참조가 2군데 발생한다. 따라서 둘 중 테이블의 외래키를 관리할 곳을 지정해야하며 외래 키를 관리하는 참조가 바로 “연관관계의 주인”이다.

연관관계의 주인이 아닌 반대편의 관계는 외래 키에 영향을 주지 않으며, 단순하게 조회만 가능하다.

<br/>

## 다대일

### 다대일 단방향

- 가장 많이 사용하는 관계

*<그림 1> 테이블과 객체의 다대일 단방향 연관관계 비교* 

![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/00ae8527-c27d-4d49-a0cd-6b3c9d557769)

다대일 연관관계에서 참조는 항상 “N”(다)에 존재해야 한다. 예를 들어 Team을 만들어 놓고 Member를 insert할 때 마다 만들어 놓은 Team을 insert하면 된다. 하지만 Team에 Member를 참조하면 Team 테이블에 여러 개 생성되는 Member마다 insert를 새롭게 해주어야 한다.  


<br/>


### 다대일 양방향

*<그림 2> 테이블과 객체의 다대일 양방향 연관관계 비교* 

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/ddf3d2df-937c-4297-b1f3-78517e00bdb8)



다대일을 **양방향 연관관계**로 만들기 위해서는 Team에 List members를 참조값으로 설정해서 Team Table에서도 join쿼리를 직접 만들 필요없이 member테이블을 조회할 수 있게 된다. 이 경우 **mappedby(”team”)를 이용하여 연관관계 주인을 Member로 설정**하였기 때문에 Team은 조회만 가능하며 테이블 변경이 불가능하여 설계상 안전하다. 

<br/>
<br/>

## 일대다

### 단방향

- @JoinColumn을 사용하지 않으면 조인 테이블 방식(중간에 테이블을 하나 추가함)

<코드 1> 추가된 조인 테이블 쿼리

```
create table Team_Member (
   Team_TEAM_ID bigint not null,
    members_MEMBER_ID bigint not null
)

```

*<그림 3> 테이블과 객체의 일대다 단방향 연관관계 비교* 

![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/59745cb2-6e6f-4804-8ad0-aa499644b0dd)



<br/>

*<그림 4> 일대다 단방향 연관관계 시 데이터 insert시 쿼리*

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/f9c71143-fe42-4367-8b6e-d9a37fee5dba)



<그림 3>처럼 일대다 단방향 연간관계에 경우 <그림 4>처럼 **추가적으로** **update 쿼리가 실행**된다. 그 이유는 Team이 연관관계의 주인이 되고 List member를 업데이트하면 DB 테이블 Member의 외래키가 NULL에서 해당 TEAM_ID로 업데이트 되기 때문이다. → 추가적인 update 쿼리의 실행으로 비용 손실. + Team에 insert를 했는데 Member에서 update가 되며 이는 일관성이 떨어짐.

- 일대다 단방향 매핑의 단점
1. 엔티티가 관리하는 외래키가 다른 테이블에 있음
2. 추가적인 update 쿼리 실행

<br/>

### 일대다 양방향

@JoinColumn(insertable = false, updateable = false) 어노테이션을 “N” 테이블에 추가한다. 이를 통해 “N” 테이블은 조회만 가능해진다. 

<br/>
<br/>

## 일대일 관계

- 주 테이블이나 대상 테이블 중에  외래 키 선택 가능
    - 주 테이블에 외래 키 (Member)
        
        장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
        
        단점 : 값이 없으면 외래 키에 null을 허용
        
    
    - 대상 테이블에 외래 키 (Team)
        
        장점 :  주 테이블과 대상 테이블을 일대다 관계로 변경할 때 테이블 구조 유지
        
        단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨.
        

- 외래 키에 데이터베이스 유니크(UNI) 제약조건을 추가해야한다.
- @OneToOne을 사용.

<br/>
<br/>

## 다대다

- 실무에서 쓰지 말아야하는 연관관계 매핑

DB 테이블은 테이블 2개로 다대다 관계가 불가능하지만 , 객체는 컬랙션을 사용해서 객체 2개로 다대다 관계가 가능하며 @ManyToMany를 통해 연결할 수 있다. 하지만 @ManyToMany은 두 객체를 연결만 할 뿐이며 주문 시간, 수량 같은 데이터가 들어올 수 없다. 

<br/>

### 다대다 한계 극복

연결 테이블용 엔티티를 추가한다(연결 테이블을 엔티티로 승격한다)

@ManyToMany → @OneToMany  + @ManyToOne

*<그림 5> 매핑테이블용 객체 엔티티를 추가한다(MemberProduct)*

![Untitled 4](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/959e6c32-52ba-42b3-9763-9672468d974c)


<br/>


*<코드 2> @ManyToOne으로 연결된 연결 테이블용 엔티티*

```python
@Entity
public class MemberPorduct {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Member product;
}

```
