---
title:  "연결 리스트 삽입과 삭제"
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


## 삽입 코드 연결 과정
<img width="655" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-04_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2 06 24" src="https://github.com/user-attachments/assets/6aa68df2-e7e2-4a0d-ba01-e8371a2a4723">
1. newNode.next를 prev.next로 연결.
2. prev.next를 newNode로 변경.
3. nodeCount를 1 증가시킨다.
- newNode 2 → 1 → 3으로 하면 안되는 걸까?
	- 당연하게도 2를 먼저해버리면 이전에 [newNode.next](http://newNode.next) 변수에 prev.next를 저장할 수 없게된다.

<br/>
<br/>

## 삽입 코드 구현 시 주의 사항

(1) 삽입하려는 위치가 리스트의 맨 앞일 경우.

	→ prev 없음

	→ Head 조정이 필요없음

<br/>

(2) 삽입하려는 위치가 리스트의 맨 끝에 있을 경우.

	→ Tail 조정이 필요하다.

	→ pos == nodeCount + 1 인 경우 맨 앞에서부터 찾아갈 필요가 없음

<br/>

(3) 빈 리스트에 삽입하는 경우.

→ (1)과 (2)의 조건을 통해 삽입이 이루어진다. 

```python
def insertAt(self, pos , newNode) : 
	if pos < 1 or pos > self.nodeCount + 1 : # pos가 올바른 범위 내에 있는지.
			return False
	
	if pos == 1 :  # 삽입하려는 위치가 리스트의 맨 앞일 경우.
		newNode.next = self.head
		self.head = newNode
	
	else : 
		if pos == self.nodeCount + 1:
			prev = self.tail
		else :
			prev = self.getAt(pos - 1)
			
		newNode.next = prev.next
		prev.next = newNode
		
	if pos == self.nodeCount + 1 : # 삽입하려는 위치가 리스트의 맨 끝에 있을 경우.
		self.tail = newNode
	
	self.nodeCount += 1
	return True
```

<br/>
<br/>

### 연결 리스트 원소 삽입의 복잡도

> 맨 앞에 삽입 : O(1)

> 중간에 삽입 : O(n)

> 맨 끝에 삽입 : O(1)

---

<br/>
<br/>

## 삭제 코드 구현 시 주의 사항

(1) 삭제하려는 node 가 맨 앞의 것일 때

	→ prev 없음

	→ Head 조정 필요

<br/>

(2) 리스트 맨 끝의 node를 삭제할 때

	→ Tail 조정 필요

	→ pos == nodeCount인 경우도 삽입과 마찬가지로 리스트를 순회할 필요가 없을까?

	→ 순회하지 않는 다면 prev를 알 방법이 없기 때문에 따로 순회 후 조정이 필요하다.

<br/>

(3) 유일한 노드를 삭제할 때

→ (1), (2)의 조건에 의해서 처리되는가? → (1) 조건에서 nodeCount가 1일 경우의 조건을 추가해준다.

```python
def popAt(self, pos):
		if pos < 1 or pos > self.nodeCount:
			raise IndexError
		
		if(pos == 1):
			curr = self.head
			self.head = self.head.next
			if (self.nodeCount == 1):
				self.tail = None
		else:       
			prev = self.getAt(pos - 1)
			curr = prev.next
			prev.next = curr.next 
			if(pos == self.nodeCount):
				self.tail = prev
				
					
		self.nodeCount -= 1
		
		return curr.data
```

<br/>
<br/>

### 연결 리스트 원소 삭제의 복잡도


> 맨 앞에 삽입 : O(1)

> 중간에 삽입 : O(n)

> 맨 끝에 삽입 : O(n)
 

---

<br/>
<br/>

## 리스트의 연결

```python
def concat(self, L):
	self.tail.next = L.head
	if L.tail : 
		self.tail = L.tail
	self.nodeCount += L.nodeCount
```

- if L.tail  조건이 없다면 어떤 exception이 일어날까?

`L`이 비어있는 리스트라면 (`L.head`와 `L.tail`이 모두 `None`), `self.tail.next = L.head` 부분이 `None.next`로 호출되어 `AttributeError`가 발생할 수 있다.
