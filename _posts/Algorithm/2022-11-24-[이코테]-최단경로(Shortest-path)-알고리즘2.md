---
title: 이코테 Ch.7 최단 경로(Shortest Path) 알고리즘 (2)
categories: [Algorithm, Shortest Path]
---

# 미래도시

N의 범위가 100이하인 전형적인 플로이드 워셜 알고리즘 문제이다. 이 문제는 1번 노드에서 K를 거쳐 X로 가는 최단거리를 구해야한다. 따라서 (1번 노드에서 K노드까지 최단 거리 + X까지 최단거리)의 값을 구하면 된다.

```python
# 무한값으로 10억
INF = int(1e9)

# 노드의 개수와 간선의 개수 입력
n, m = map(int, input().split())
# 2차원 그래프 생성하고 모든 값을 무한으로 초기화
graph = [[INF] * (n+1) for _ in range(n+1)]

# 자기 자신에게 가는 거리는 0으로 초기화
for a in range(1, n+1):
  for b in range(1, n+1):
    if a == b:
      graph[a][b] = 0

# 각 간선에 대한 정보를 입력 받아 1로 초기화
for _ in range(m):
  a, b = map(int, input().split())
  graph[a][b] = 1
  graph[b][a] = 1

# 거치갈 노드와 최종 목적지 노드 입력
x, k = map(int, input().split())

# 점화식에 따라 플로이드 워셜 알고리즘을 수행
for k in range(1, n+1):
  for a in range(1, n+1):
    for b in range(1, n+1):
      graph[a][b] = min(graph[a][b], graph[a][k] + graph[k][b])

# 수행된 결과를 출력
distance = graph[1][k] + graph[k][x]

# 거리가 무한대이면 -1 출력, 아니면 거리값 출력
if distance >= INF:
  print("-1")
else:
  print(distance)
```

# 전보
n개의 도시는 통로로 연결되어 있다면, 전보를 보낼 수 있다. 다만 통로는 일방향이고, 따라서 두 도시가 서로 전보를 주고 받기 위해서는 두 방향의 통로가 모두 있어야한다. 이때 도시 C에서 보낸 메세지를 받게 되는 도시의 개수는 총 몇개이며 도시들이 모두 메세지를 받는데 까지 걸리는 시간은 얼마인지 계산하는 건 도시간의 최단 거리문제로 치환해도 가능하다. 즉, 다익스트라 알고리즘을 이용해서 풀 수 있다는 것이다.

n과 m의 범위가 충분히 크기 때문에 우선순위 큐를 이용하여 다익스트라 알고리즘을 작성해야 하며, 실질적으로 작성해야하는 코드는 앞선 포스팅의 예제들에서 후반부만 수정해주는 수준이다.

```python
import heapq
import sys

input = sys.stdin.readline
INF = int(1e9)

# 노드 개수, 간선 개수, 시작 노드 입력
n, m, start = map(int, input().split())
# 각 노드에 연결되어 있는 노드에 대한 정보를 담는 리스트 만들기
graph = [[] for i in range(n+1)]
# 최단 거리 테이블을 모두 무한으로 초기화
distance = [INF] * (n+1)

# 모든 간선 정보를 입력받기
for _ in range(m):
  x, y, z = map(int, input().split())
  # x번 노드에서 y번 노드로 가는 비용이 z
  graph[x].append((y, z))

def dijkstra(start):
  q = []
  # 시작 노드로 가기 위한 최단 경로는 0으로 설정하고 큐에 삽입
  heapq.heappush(q, (0, start))
  distance[start] = 0
  # 큐가 비어있지 않다면 반복
  while q:
    # 가장 최단 거리가 짧은 노드에 대한 정보를 꺼내기
    dist, now = heapq.heappop(q)
    if distance[now] < dist:
      continue
    # 현재 노드와 연결된 다른 인접한 노드들을 확인
    for i in graph[now]:
      cost = dist + i[1]
      # 현재 노드를 거쳐서, 다른 노드로 이동하는 거리가 더 짧은 경우
      if cost < distance[i[0]]:
        distance[i[0]] = cost
        heapq.heappush(q, (cost, i[0]))

dijkstra(start)

# 도달할 수 있는 노드의 수
count = 0
# 도달할 수 있는 노드들 중, 가장 멀리 있는 노드와의 최단거리
max_distance = 0
for d in distance:
  if d != INF:
    count += 1
    max_distance = max(max_distance, d)

# 시작노드는 제외하기 위해 count에서 -1을 해줌
print(count - 1, max_distance)
```
