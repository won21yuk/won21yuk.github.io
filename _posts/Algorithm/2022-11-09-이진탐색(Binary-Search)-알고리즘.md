---
title: 이코테 Ch.5 이진탐색(Binary search) 알고리즘 (1)
categories: [Algorithm, Binary search]
---

# 순차탐색

순차 탐색은 리스트에서 특정한 데이터를 찾기 위해 앞에서 부터 순차적으로 하나씩 확인하는 탐색방법이다.

보통 정렬이 되지 않은 리스트에서 데이터를 찾아야하는 경우 사용한다.

```python
def sequential_search(n, target, array):
	for i in range(n):
		if array[i] == target:
			return i + 1

print('생성할 원소 개수를 입력한 후에 한칸 띄고 찾을 문자열을 입력하세요')
input_data = input().split()
n = int(input_data[0])
target = input_data[1]

print('앞서 적은 원소 개수만큼 문자열을 입력하세요. 구분은 띄어쓰기 한 칸으로 합니다')
array = input().split()

print(sequential_search(n, target, array))
```

리스트에서 특정 원소를 찾을 때 그리고 원소의 개수를 셀 때도 내부적으로는 순차 탐색이 작동한다.

순차 탐색은 무조건 가장 앞에서부터 탐색하기 때문에 데이터가 N개 일때 최악의 경우 O(N)의 시간 복잡도를 가진다.

# 이진탐색

이진 탐색은 배열의 데이터들이 정렬되어 있어야만 사용할 수 있는 알고리즘이다.

따라서 정렬이 안되어있으면 시간이 매우 오래걸리지만, 정렬이 되어있다면 매우 빠른 속도로 데이터를 탐색할 수 있다.

이진탐색은 이름에서 보다시피 범위를 절반씩 줄여가며 탐색하는 방법이다. 그래서 기본적으로 전체 배열의 위치정보를 표시하기위해 시작점, 중간점, 끝점이라는 세 개의 변수를 사용한다.

이진 탐색은 찾으려는 데이터와 중간점 위치에 있는 데이터를 반복적으로 비교해서 원하는 데이터를 찾아간다.

이진탐색은 알고리즘을 한번 거칠 때마다 배열의 범위가 반으로 줄어들기 때문에 시간복잡도가 O(logN)이다.

이진탐색은 구현하는 방법에는 재귀함수를 이용하는 방법과 단순 반복문을 이용하는 방법 두가지가 있다.

우선 재귀함수를 통해 구현한 이진탐색 코드는 아래와 같다.

```python
def binary_search(array, target, start, end):
	if start > end: # 배열의 범위를 벗어난다면 None 반환
		return None
	mid = (start+end) // 2 #소수점은 버리기위해 몫을 중간값으로 한다
	if array[mid] == target:
		return mid
	# target이 mid값보다 작으면 끝점이 mid -1가 되도록 범위 줄이기
	elif array[mid] > target:
		return binary_search(array, target, start, mid -1)
	# target이 mid값보다 크면 시작점이 mid+1이 되도록 범위 줄이기
	else:
		return binary_search(array, target, mid + 1, end)

n , target = list(map(int, input().split()))
array = list(map(int, input().split()))

result = binary_search(array, target, 0, n-1)
if result == None:
	print('원소가 존재하지 않습니다.')
else:
	print(result+1)

```

반복문을 사용한 이진탐색 코드는 아래와 같다.

```python
def binary_search(array, target, start, end):
	while start <= end:
		mid = (start + end) // 2
		if array[mid] == target:
			return mid
		elif array[mid] > target:
			end = mid - 1
		else:
			start = mid + 1
	return None



n , target = list(map(int, input().split()))
array = list(map(int, input().split()))

result = binary_search(array, target, 0, n-1)
if result == None:
	print('원소가 존재하지 않습니다.')
else:
	print(result+1)

```

# 코딩 테스트에서의 이진 탐색

이진탐색은 막상 아무것도없는 상태에서 작성하려면 훨씬 어렵다. 그런데 또 코딩테스트에서는 단골 문제이다. 가급적 반복 연습을 통해 암기하는것이 좋다.

코딩 테스트에서 이진탐색 문제는 보통 탐색범위가 큰 상황에서의 탐색을 가정하는 문제가 많다. 따라서 탐색범위가 2000만을 넘어가면 이진탐색으로 접근해보면 좋다

# 트리 자료구조

트리 자료구조는 노드와 노드의 연결로 표현하며, 여기에서 노드는 정보의 단위로써 어떤 정보를 가지고 있는 개체로 이해할 수 있다.

트리 자료구조는 그래프 자료구조의 일종으로 데이터베이스 시스템이나 파일 시스템과 같은 곳에서 많은 양의 데이터를 관리하기 위한 목적으로 사용한다.

트리 자료구조의 특징은 아래와 같다.

- 트리는 부모 노드와 자식 노드의 관계로 표현
- 트리의 최상단 노드는 루트 노드
- 트리의 최하단 노드는 단말 노드
- 트리에서 일부를 떼어내도 트리 구조이며, 이를 서브 트리라 부름
- 트리는 파일 시스템과 같이 계층적이고 정렬된 데이터를 다루기에 적합

정리하면 큰 데이터를 처리하는 소프트웨어는 대부분 데이터를 트리 자료구조로 저장해서 이진 탐새과 같은 탬색 기법을 이용해 빠르게 탐색이 가능하다.

# 이진 탐색 트리

트리 구조중 가장 간단한 형태가 이진 탐색 트리다.

이진 탐색의 특징은 아래와 같다.

- 부모 노드보다 왼쪽 자식 노드가 작다
- 부모 노드보다 오른쪽 자식 노드가 크다

이진 탐색 트리에 데이터를 넣고 빼는 작업은 알고리즘보단 자료구조에 더 가깝다. 다만 이진 탐색 트리 자료구조 자체를 구현하는 문제는 출제되지 않는다.

그래서 중요한건 이진 탐색 트리가 주어진 상태에서 데이터를 탐색하는 것이다.

![bs1](/images/bs1.png)

위의 그림을 통해 이진 탐색 트리에서 탐색 이떻게 이루어지는지 보도록하겠다.

1. 기본적으로 이진 탐색은 루트 노드부터 방문한다.

1. 내가 찾고자 하는 데이터가 루트 노드의 값보다 큰지 작은지를 판별한다.

1. 14는 루트 노드의 값인 10보다 크기때문에 오른쪽 노드를 바로 방문한다.

1. 이젠 오른쪽 자식 노드인 17이 부모 노드다. 14는 17보다 작기때문에 왼쪽 노드를 방문한다.

1. 그리고 현재 방문한 노드가 14와 같으니 탐색을 마친다.

# 빠르게 입력 받기

이진 탐색 문제는 보통 천만개 이상의 데이터 범위에서 탐색을 하도록 주어지기 때문에 이전의 알고리즘 문제들처럼 input으로 입력하면 시간이 너무 오래걸린다.

이때는 sys라이브러리의 readline 함수를 이용해서 입력하면 된다.

```python
import sys
input_data = sys.stdin.readline().rstrip()

print(input_data)
```

여기서 주의 해야할 건 반드시 rstrip 함수를 써줘야한다는 점이다. readline으로 입력하면 입력후 엔터를 치면 줄바꿈 기호로 인식하기 때문에 이 공백문자를 rstrip으로 제거해주어야 한다.
