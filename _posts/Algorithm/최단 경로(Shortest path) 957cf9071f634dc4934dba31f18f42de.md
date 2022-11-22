# 최단 경로(Shortest path)

최단 경로 알고리즘은 가장 짧은 경로를 찾는 알고리즘을 의미한다. 코딩테스트에서 사용되는 최단 경로 알고리즘의 종류는 다익스트라와 플로이드 워셜 알고리즘이 있다.

# 다익스트라 최단 경로 알고리즘

특정한 노드에서 출발하여 다른 모든 노드로 가는 최단 경로를 계산하는 알고리즘이다. 그리디 알고리즘의 한 종류로 분류되기 때문에 매 상황에서 가장 비용이 적은 노드를 선택해 임의의 과정을 반복하는 구조이다. 자세한 동작과정은 아래와 같다.

1. 출발 노드를 설정한다
2. 최단 거리 테이블을 초기화한다.
3. 방문하지 않은 노드 중에서 최단 거리가 가장 짧은 노드를 선택한다.
4. 해당 노드를 거쳐 다른 노드로 가는 비용을 계산하여 최단 거리 테이블을 갱신한다.
5. 위 과정에서 3번과 4번을 반복한다.

```python
import sys
input = sys.stdin.readline
INF = int(1e9) # 무한을 의미하는 값으로 10억을 설정

# 노드의 개수, 간선의 개수 
n, m = map(int, input().split())
# 시작 노드 번호 입력
start = int(input())
# 각 노드에 연결되어 있는 노드에 대한 정보를 담는 리스트 만들기
graph = [[] for i in range(n+1)]
# 방문한 적이 있는지 체크하는 목적의 리스트 만들기
visited = [False] * (n+1)
# 최단 거리 테이블을 모두 무한으로 초기화
distance = [INF] * (n+1)

# 모든 간선 정보 입력
for _ in range(m):
	a, b, c = map(int, input().split())
	# a번 노드에서 b번 노드로 가는 비용은 c
	graph[a].append((b,c))

# 방문하지 않은 노드 중에서 가장 최단 거리가 짧은 노드의 번호를 반환
def get_smallest_node():
	min_value = INF
	# 가장 최단 거리가 짧은 노드(인덱스)
	index = 0 
	for i in range(1, n+1):
		if distance[i] < min_value and not visited[i]:
			min_value = distance[i]
			index = i
	return index

def dijkstra(start):
	# 시작 노드에 대해서 초기화
	distance[start] = 0
	visited[start] = True
	for j in graph[start]:
		distance[j[0]] = j[1]
	# 시작 노드를 제외한 전체 n-1개의 노드에 대해 반복
	for i in range(n-1):
		# 현재 최단거리가 가장 짧은 노드를 꺼내서, 방문처리
		now = get_smallest_node()
		visited[now] = True
		# 현재 노드와 연결된 다른 노드 확인
		for j in graph[now]:
			cost = distance[now] + j[1]
			# 현재 노드를 거쳐서 다른 노드로 이동하는 거리가 더 짧은 경우
			if cost < distance[j[0]]:
				distance[j[0]] = cost

dijkstra(start)

# 모든 노드로 가기 위한 최단 거리 출력
for i in range(1, n+1):
	if distance[i] == INF:
		print("INFINITY")
	else:
		print(distance[i])

```

위와 같이 코드를 구성하면 총 O(V)번에 걸쳐서 최단 거리가 가장 짧은 노드를 매번 선형 탐색해야 하기 때문에 전체 시간 복잡도는 O(V^2)이다. 여기서 V는 노드의 개수를 의미한다. 일반적으로 코딩 테스트의 최단 경로 문제에서 전체 노드의 개수가 5000개 이하라면, 이 코드가 문제가 없지만 노드의 개수가 10000개가 넘어가면 이 코드로 문제를 해결하기 어렵다. 따라서 노드의 개수가 10000개가 넘어가면 개선된 다익스트라 알고리즘을 사용해야한다.

## 개선된 다익스트라 알고리즘

이 개선된 방식을 사용하면, 시간복잡도는 O(ElogV)를 가질 수 있다. 여기서 V는 노드의 개수이고 E는 간선의 개수이다.

개선된 다익스트라 알고리즘은 힙(Heap) 자료구조를 사용한다. 힙 자료구조는 우선순위 큐를 구현하기 위하여 사용하는 자료구조 중 하나이다. 우선순위 큐는 우선순위가 가장 높은 데이터를 가장 먼저 삭제한다는 점이 특징이다. 다행이도 파이썬에서는 우선순위 큐 라이브러리인 heapq를 제공하기 때문에 이를 사용하면 된다.

또한 우선순위 큐를 구현할 때는 최소 힙 혹은 최대 힙을 이용한다. 최소힙을 이용하는 경우 값이 낮은 데이터가 먼저 삭제되며 최대 힙을 이용하는 경우 값이 큰 데이터가 먼저 삭제된다. heapq 라이브러리는 최소 힙 구조를 이용하는데 다익스트라 최단 경로 알고리즘도 비용이 적은 노드를 우선하여 방문하므로 그대로 사용하면 된다.

개선된 다익스트라 알고리즘에서는 단순히 우선순위 큐를 이용해서 시작 노드로부터 거리가 짧은 노드 순서대로 큐에서 나올 수있도록 코드를 작성하면 된다. 간단히 말하면, 현재 가장 가까운 노드를 저장하기 위한 목적으로만 우선순위 큐를 추가로 이용한다는 것 이다. 

힙 라이브러리를 사용한 예제 코드는 아래와 같다.

```python
import heapq

def heapsort(iterable):
	h = []
	result = []
	for value in iterable:
		heapq.heappush(h, value)
	for i in range(len(h)):
		result.append(heapq.heappop(h))
	return result

result = heapsort([1, 3, 5, 7, 9, 2, 4, 6, 8])
print(result)
```

앞서 말했듯 heapq라이브러리는 최소 힙을 디폴트로 하고 있기 때문에 만약 최대 힙을 사용하고 싶다면 heap에 값을 집어넣을 때 -로 부호를 바꿔주고 꺼낼때 다시 -를 붙여주면 최대힙의 형태로 사용할 수있다. 그렇게 작성한 코드는 아래와 같다.

```python
import heapq

def heapsort(iterable):
	h = []
	result = []
	for value in iterable:
		heapq.heappush(h, -value)
	for i in range(len(h)):
		result.append(-heapq.heappop(h))
	return result

result = heapsort([1, 3, 5, 7, 9, 2, 4, 6, 8])
print(result)
```

결국 힙을 사용하여 개선시킨 다익스트라 알고리즘을 코드로 구현하면 아래와 같다.

```python
import heapq
import sys
input = sys.stdin.readline
INF = int(1e9) # 무한을 의미하는 값으로 10억을 설정

# 노드의 개수, 간선의 개수 
n, m = map(int, input().split())
# 시작 노드 번호 입력
start = int(input())
# 각 노드에 연결되어 있는 노드에 대한 정보를 담는 리스트 만들기
graph = [[] for i in range(n+1)]
# 최단 거리 테이블을 모두 무한으로 초기화
distance = [INF] * (n+1)

# 모든 간선 정보 입력
for _ in range(m):
	a, b, c = map(int, input().split())
	# a번 노드에서 b번 노드로 가는 비용은 c
	graph[a].append((b,c))

def dijkstra(start):
	q = []
	# 시작 노드로 가기 위한 최단 경로 0으로 설정하여 큐에 삽입
	heapq.heappush(q, (0, start))
	distance[start] = 0
	while q: # 큐가 비어있지 않으면 무한 루프
		# 가장 최단 거리가 짧은 노드에 대한 정보 꺼내기
		dist, now = heapq.heappop(q)
		# 현재 노드가 이미 처리된 적이 있는 노드라면 무시
		if distance[now] < dist:
			continue
		# 현재 노드와 연결된 다른 인접한 노드들을 확인
		for i in graph[now]:
			cost = dist + i[1]
			# 현재 노드를 거쳐서 다른 노드로 이동하는 거리가 더 짧은 경우
			if cost < distance[i[0]]:
				distance[i[0]] = cost
				heapq.heappush(q, (cost, i[0]))

dijkstra(start)

# 모든 노드로 가기 위한 최단 거리를 출력
for i in range(1, n+1):
	if distance[i] = INF:
		print("INFINITY")
	else:
		print(distance[i])
```

노드를 하나씩 꺼내 검사하는 while 문은 노드의 개수 V 이상의 횟수로는 처리 되지 않는다. 결과적으로 현재 우선숭위 큐에서 꺼낸 노드와 연결된 다른 노드들을 확인하는 총 횟수는 최대 간선의 개수 E만큼 연산이 수행된다. 

직관적으로 보면 전체 과정은 E개의 원소를 우선순위 큐에 넣었다가 모두 빼내는 연산과 유사하다. 그렇게 때문에 시간 복잡도를 O(ElogE)로 판단할 수 있다. 이 문제처럼 중복 간선을 포함하지 않는 경우에는 이를 O(ElogV)로 정리할 수 있다.

# 플로이드 워셜 알고리즘

이 알고리즘은 모든 노드에서 다른 모든 노드까지의 초단 경로를 모두 계산한다. 다익스트라 알고리즘과 마찬가지로 단계별로 거쳐 가는 노드를 기준으로 알고리즘을 수행하지만, 매 단계마다 방문하지 않는 노드 중에 최단 거리를 갖는 노드를 찾는 과정이 필요하진 않다. 또한 플로이드 워셜 알고리즘을 2차월 테이블을 사용해 최단 거리 정보를 저장하며, 다이나믹 프로그래밍 유형에 속한다.

플로이드 워셜 알고리즘은 각 단계바다 특정한 노드 K를 거쳐가는 경우를 확인하는데 가령 A에서 B로 가는 최단거리보다 A에서 K를 거쳐 B로 가는 거리가 더 짧은 지를 검사한다. 따라서 점화식은 아래와 같다.

![Untitled](%E1%84%8E%E1%85%AC%E1%84%83%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A7%E1%86%BC%E1%84%85%E1%85%A9(Shortest%20path)%20957cf9071f634dc4934dba31f18f42de/Untitled.png)

각 노드마다 모든 경우에 대한 점화식을 확인하기 때문에 다중 반복문을 통해 플로이드 워셜 알고리즘이 구현된다. 이를 활용하여 코드로 구현하면 아래와 같다.

```python
INF = int(1e9)

n = int(input())
m = int(input())
# 2차원 리스트 만들고 무한으로 초기화
graph = [[INF] * (n-1) for _ in range(n+1)]

# 자기 자신에서 자기 자신으로 가는 비용은 0으로 초기화
for a in range(1, n+1):
	for b in range(1, n+1):
		if a == b:
			graph[a][b] = 0

# 각 간선에 대한 정보를 입력받아 그 값으로 초기화
for _ in range(m):
	# A에서 B로 가는 비용은 C
	a, b, c = map(int, input().split())
	graph[a][b] = c

# 점화식에 따라 플로이드 워셜 알고리즘 수행
for k in range(1, n+1):
	for a in range(1, n+1):
		for b in range(1, n+1):
			graph[a][b] = min(graph[a][b], graph[a][k] + graph[k][b])

# 수행된 결과를 출력
for a in range(1, n+1):
	for b in range(1, n+1):
		if graph[a][b] == INF:
			print("INFINITY", end=" ")
		else:
			print(graph[a][b], end=" ")
	print()
```

노드의 개수가 N개 일때 알고리즘상으로 N번의 단계를 수행한다. 각 단계마다 O(N^2)의 연산을 통해 현재 노드를 거쳐가는 모든 경로를 고려한다. 따라서 플로이드 워셜 알고리즘의 총 시간 복잡도는 O(N^3)이다. 이때문에 플로이드 워셜 알고리즘이 사용되어야하는 문제에서는 500개 이하의 노드가 주어지게 된다.