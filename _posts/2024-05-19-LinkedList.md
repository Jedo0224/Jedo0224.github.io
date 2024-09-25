---
title:  "연결 리스트(Linked List)"
excerpt: "어서와 자료구조와 알고리즘은 처음이지?"

categories:
  - Algorithm
tags:
  - [Python, Algorithm, Coding=Test]

toc: true
toc_sticky: true
 
date: 2024-09-24
last_modified_at: 2024-09-19
---


---

## 추상적 자료구조

- Data
    - 정수, 문자열, 레코드
- A set of operations
    - 삽입, 삭제 , 순회
    - 정렬, 탐색…

    Linked List는 존재하는 데이터를 화살표로 연결해서 관리하는 데이터 구조이다. 즉, 각 요소가 데이터와 요소를 참조(링크 , 화살표)하는 정보를 포함하는 노드로 구성된다. 이 때문에 동적인 A set of operations를 상대적으로 쉽게 할 수 있다는 장점이 있다.

---

<br/>
<br/>

## 기본적 연결 리스트 자료구조 정의

- 단일 노드

```jsx
class Node:
	def __init__(self, item):
		self.data = item
		self.next = None
```

<br/>
<br/>

- 각 단일 노드를 연결한 연결 리스트의 자료구조

```python
class LinkedList:
	def __init__(self):
		self.nodeCount = 0
		self.head = None
		self.tail = None
		
# 노드 생성 및 연결 리스트에 삽입 예시
linked_list = LinkedList()
node1 = Node(10)
node2 = Node(20)
node3 = Node(30)

# 노드를 연결 리스트에 삽입
linked_list.insertAt(1, node1)  # 첫 번째 위치에 삽입
linked_list.insertAt(2, node2)  # 두 번째 위치에 삽입
linked_list.insertAt(3, node3)  # 세 번째 위치에 삽입

# 연결 리스트 순회 및 출력
print(linked_list.traverse())  # 출력: [10, 20, 30]
```

<br/>
<br/>

### 배열과 연결리스트 비교

|  | 배열  | 연결 리스트 |
| --- | --- | --- |
| 저장 공간  | 연속한 위치 | 임의의 위치 |
| 특정 원소 지칭 | 매우 간편 | 선형탐색과 유사 |
| 특정 원소 지칭 시 시간복잡도 | O(1) | O(n) |
