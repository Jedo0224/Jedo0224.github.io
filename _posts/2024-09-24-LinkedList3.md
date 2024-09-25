---
title:  "양방향 연결리스트"
excerpt: "어서와 자료구조와 알고리즘은 처음이지?"

categories:
  - Algorithm
tags:
  - [Python, Algorithm, Coding-Test]

toc: true
toc_sticky: true
 
date: 2024-09-24
last_modified_at: 2024-09-25
---

## 더미 양방향 연결리스트 구조

```python
class Node:

	def __init__(self, item):
		self.data = item
		self.prev = None
		self.next = None

class DoublyLinkedList:

	def __init__(self):
		self.nodeCount = 0
		self.head = Node(None)
		self.tail = Node(None)
		self.head.prev = None
		self.head.next = self.tail
		self.tail.prev = self.head
		self.tail.next = None
```

![%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5 19 20](https://github.com/user-attachments/assets/d08fb91b-6e4b-4fbb-95bd-2ee96abe5fa7)

## 더미 양방향 연결리스트의 장점

- 삽입과 삭제가 보다 유연하게 이루어질 수 있다는 장점.
	
	→ 하지만 실제로  “몇 번째(pos)” 위치에 “newNode”를 삽입/삭제하라는 것을 구현하는데 있어, “몇 번째(pos)”에 “원래 어떤 노드”가 있고 그 노드에 앞 또는 뒤 링크가 무엇인지 알기 위해서는 처음부터 링크를 순회해야했다. 
	
	→ 성능을 개선하기 위해, “몇 번째(pos)” 위치의 노드에서 어떤 노드를 삽입/삭제 하는지가 아니라, 해당 노드 앞 또는 뒤 바로 삽입하도록 변경해보자. 
	
	> insert(idx,newNode) → insertAfter(prev, newNode)
	pop(idx)→ popAfter(prev)
	> 

<br/>
<br/>
	
- 순방향(앞, 다음)과 역방향(뒤, 이전)으로 모두 진행이 가능하다. 따라서 앞과 뒤 링크만 조정해주면 되기 때문에 쉽고 빠르다.
	
	→ 맨 뒤에 있는 노드를 삭제할 경우 tail.prev를 순회없이 쉽게 알 수 있다.
	
	- 역방향의 리스트 순회가 가능하다.
   
	![%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-09-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5 19 47](https://github.com/user-attachments/assets/25b3a60f-94cf-4165-94da-027a1a74fc7e)

  

```python
class Node:

	def __init__(self, item):
		self.data = item
		self.prev = None
		self.next = None

class DoublyLinkedList:

	def __init__(self):
		self.nodeCount = 0
		self.head = Node(None)
		self.tail = Node(None)
		self.head.prev = None
		self.head.next = self.tail
		self.tail.prev = self.head
		self.tail.next = None

	def concat(self, L):

		L1_last= self.tail.prev
		L2_first = L.head.next

		L1_last.next = L2_first
		L2_first.prev = L1_last

		self.tail = L.tail
		self.tail.prev = L.tail.prev         
		   
		self.nodeCount = self.nodeCount + L.nodeCount
		
		
	def traverse(self):
		result = []
		curr = self.head
		while curr.next.next:
			curr = curr.next
			result.append(curr.data)
		return result

	
	def getAt(self, pos):
		if pos < 0 or pos > self.nodeCount:
			return None

		if pos > self.nodeCount // 2:
			i = 0
			curr = self.tail
			while i < self.nodeCount - pos + 1:
				curr = curr.prev
				i += 1
		else:
			i = 0
			curr = self.head
			while i < pos:
				curr = curr.next
				i += 1

		return curr

	def insertAfter(self, prev, newNode):
		next = prev.next
		newNode.prev = prev
		newNode.next = next
		prev.next = newNode
		next.prev = newNode
		self.nodeCount += 1
		return True

	def insertAt(self, pos, newNode):
		if pos < 1 or pos > self.nodeCount + 1:
			return False

		prev = self.getAt(pos - 1)
		return self.insertAfter(prev, newNode)

def solution(x):
	return 0
```

	위치(pos)를 사용한 경우보다 적은 조건문을 이용하여 간단하고 유연한 삽입/삭제/병합이 가능해졌다.
