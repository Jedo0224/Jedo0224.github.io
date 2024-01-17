---
title:  "복합 키: 키 처리 방법에 대한 가이드"
excerpt: "[복합 키: 키 처리 방법에 대한 가이드](https://hackernoon.com/composite-keys-a-guide-on-how-to-handle-them)"

categories:
  - Java
tags:
  - [Java]

toc: true
toc_sticky: true
 
date: 2024-01-16
last_modified_at: 2024-01-17
---




## 복합키

복합 키(Composite Key)는 두 개 이상의 컬럼을 결합하여 테이블에서 레코드를 고유하게 식별하는 데 사용되는 키이다.

- 복합키를 사용하는 경우
    - map 또는 Cache를 조회할 때 “키”를 정의하기 위해 데이터 조합이 필요한 경우이다. (사용자 이름에 따란 고유한 값을 저장하는 경우 등등,,)

<br/>
<br/>
<br/>
## 문자열 결합

### 문제점

```java
//1 
private String getMapKey(Long userId, String userLocale) {
    return userId + "." userLocale;
}

```

1. 이러한 문자열의 할당은 당신이 얼마나 많은 키를 갖고 있는지에 크게 의존할 것이다. 
    예를 들면 map에 자주 접근한다면 많은 수의 할당이 이루어지고 garbage collected이 필요해 질 수 있다.
    

```java
//2 
public String getMapKey(Integer groupId, Integer accessType) {
    return groupId.toString() + accessType.toString();
}
```

1. 내가 생성한 복합키가 다른 키값에 spoofing(같은 사용자인 것처럼 위장되는 것)/되면 안된다.
    groupId = 1 and accessType = 23, groupId = 12 and accessType = 3의 각 문자열을 그냥 합쳐져 123이 되어 overlap되지 않도록 “,”를 추가하는 등의 작업을 추가해야한다.

```java
//3
public String getMapKey(String userProvidedString, String extensionName) {
    return userProvidedString + (extensionName == null ? "" : ("." + extensionName));
}
```

extensionName 키가 선택적이다. 이 경우 생길 수 있는 문제점은 예를 들어 **`userProvidedString`**이 "data.txt"이고 **`extensionName`**이 **`null`**이라면, 이 함수는 "data.txt"라는 키를 반환한다. 하지만 **`userProvidedString`**이 "data"이고 **`extensionName`**이 "txt"라면, 함수는 "data.txt"라는 동일한 키를 반환한다. 이런 상황에서, 두 번째 경우에서 사용자가 실제로 "data.txt"라는 이름의 캐시 데이터에 접근할 수 있게 되어 보안 및 데이터 무결성에 문제가 될 수 있다. 



<br/><br/><br/>
## Use Nested Maps/Caches

Maps of Maps of Maps와 같은 복잡한 자료구조를 이용하는 방법.

```java
Map<Integer, Map<String, String>> groupAndLocaleMap = new HashMap<>();
groupAndLocaleMap.computeIfAbsent(userId, k -> new HashMap()).put(userLocale, mapValue);
```

### 장점

Map으로 전달된 키 값이 이미 할당되어있기 때문에 내부의 MAP과 상호작용 할 경우 새 메모리를 할당할 필요가 없다. 

### 문제점

1. 3개 이상의 데이터를 처리하기 시작하면 코드 가독성이 낮아지고 및 맵 초기화가 어려워진다. 또한 각 레벨에서는 null point exception을 피하기 위한 검사가 필요하다. 

### 따라서 문자열 결합과 ****Nested Maps/Caches 모두 갖고 있는 키의 고유성에 따라 공간 효율성이 떨어진다고 볼 수 있다.****


<br/><br/><br/>

## Create a Composite Key Object

```java
private class MapKey {  
    private final int userId;  
    private final String userLocale;  

    public MapKey(int userId, String userLocale) {  
        this.userId = userId;  
        this.userLocale = userLocale;  
    }  

    @Override  
    public boolean equals(Object o) {  
        if (this == o) return true;  
        if (o == null || getClass() != o.getClass()) return false;  
        MapKey mapKey = (MapKey) o;  
        return userId == mapKey.userId && Objects.equals(userLocale, mapKey.userLocale);  
    }  

    @Override  
    public int hashCode() {  
        return Objects.hash(userId, userLocale);  
    }  
}
```

### 장점

각 키를 구성하는 부분을 문자열로 다시 할당할 필요가 없이 객체 키에만 새 메모리만 필요하기에 문자열 보다 메모리 효율에 이점이 있다. 

key equality and hashcode implementations에 대한 커스터마이징이 가능하다. (ex . 문자열에서 대문자 무시, 배열이나 컬랙션을 key값으로 사용하는 등)

이해하기 쉽고 최소한의 코드를 요구하며 나중에 디버깅하기 쉽다. 

### 단점

1. 객체 생성 오버헤드가 발생할 수 있다. 


<br/><br/><br/>

## 성능 비교

### Retained Map Size

- 100 ints, 100 strings, 100 longs — 1 million unique keys  (각 한개의 키에  100 ints, 100 strings, 100 longs 이 들어가있음)
- 1 int, 1 string, 1,000,000 longs— 1 million unique keys
- 1,000,000 ints, 1 string, 1 long — 1 million unique keys

![Untitled](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/112a587a-fafe-40fe-83a1-c64885e562ba)


- heap size가 nested Maps에서 크게 증가함.
    - nested Maps
        - 1번 시나리오
            
            100 x 100 x 100 = 1,000,000개의 조합이 만들어진다. 여기에 최상위 맵 100개를 더하면 총 1,000,100개의 맵이 만들어진다.
            
        - 2번 시나리오
            
            1 x 1 x 1,000,000 = 1,000,000 , 여기에 최상위 맵 1개를 더하여 1,000,001이 만들어진다.
            
        - 3번 시나리오
            
            1,000,000 x 1 x 1= 1,000,000 , 여기에 최상위 맵 1,000,000개를 더하여 2,000,000이 만들어진다.
            
    

### Creation Duration & Creation Memory Allocation (MB)

![Untitled 1](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/2323fd1c-a3d0-4205-ad6e-2ad2b5c59004)

![Untitled 2](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/d8726cfc-f114-4fbf-9b2a-cde9298a785a)



- nested Maps 메모리 할당이 크지만 CPU 시간에서 다른 맵보다 성능이 뛰어남.
- 문자열 결합은 nested Maps 보다 약간 느리고 Composite Key보다 더 많은 메모리 할당이 필요.




## Lookup Duration & Lookup Allocation

![Untitled 3](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/e198541a-183d-48ef-8680-62d0e4e068f1)


![Untitled 4](https://github.com/Jedo0224/Jedo0224.github.io/assets/90050514/42a0f759-ea21-4041-93be-e182fcd1bc80)


- 문자열 결합 기반의 키조회가 가장 느림을 알 수 있으며 **nested Maps에서는 조회를 수행하기 위한 추가적 메모리 할당이 필요하지 않다.**
    
    
    - **nested Maps에서는 조회를 수행하기 위한 추가적 메모리 할당이 필요치 않은 이유.**
    
    ```java
    private String lookup(int key1, String key2, long key3) {
        return map.get(key1).get(key2).get(key3);
    }
    
    // composite key
    private String lookup(int key1, String key2, long key3) {
        return map.get(new MapKey(key1, key2, key3)); 
    // 새 객체를 할당하지만 해당 객체의 멤버가 이미 생성 및 참조 되어있어
    // 문자열 보다는 적은 양을 할당 
    }
    ```
    
    주어진 부분을 각 nested map의 키로 재사용하기 때문이다.
    

<br/><br/><br/>


## 결론

자체 테스트를 수행하여 서비스 성능에 알맞은 방식을 선택해야한다.

이 때 Composite key는 Nested maps가 사용 불가능할 때 매우 유용한 선택지가 될 수 있다.

또한  Composite key와 Nested maps를 합쳐서 사용할 수 있는 방법이 있는데. 예를 들어 첫번째 또는 두 번째 수준에 Nested maps을 사용하고 그 이후 더 깊은 수준을 Composite key를 사용하여 단순화 시킬 수 있다.

Composite key를 통해 스토리지 및 조회 성능을 최적화 하는 동시에 빠른 조회를 위해 데이터를 분할 된 상태로 유지하고 코드 가독성 또한 높일 수 있다.

<br/><br/>
### reference 
https://hackernoon.com/composite-keys-a-guide-on-how-to-handle-them
