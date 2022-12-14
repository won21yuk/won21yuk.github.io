---
title: 이코테 Ch.6 DP(Dynamic Programming) 알고리즘 (3)
categories: [Algorithm, Dynamic Programming]
---
# 바닥 공사

가로의 길이가 N, 세로의 길이가 2인 바닥을 덮개로 모두 덮는 경우의 수 구하는 문제이다. 덮개의 가지수는 총 3개이고 아래와 같다.

- 1 X 2
- 2 X 1
- 2 X 2

```python
"""
입력조건 : 첫째 줄에 N이 주어진다.
출력조건 : 첫째줄에 2XN 크기의 바닥을 채우는 방법의 수를 796,796으로 나눈 나머지 출력
=> 결과값이 매우 커질 수 있기 때문에 특정한 수로 나눈 나머지를 취하게 하는 것임
"""
n = int(input())

d = [0]*1001

d[1] = 1
d[2] = 3

for i in range(3, n+1):
	d[i] = (d[i-1] + 2*d[i-2])%796796

print(d[n])
```

# 효율적인 화폐구성

n개의 종류의 화폐가 있을 때, 이 화폐들의 개수를 최소한으로 용해서 그 가치의 합이 m원이 되도록하려 한다.

n=3, m=7이고 각 화폐 단위가 2,3,5인 경우를 가정해본다.

우선 각 인덱스에 해당하는 값으로 무한대의 값을 설정한다. 무한대의 값은 화폐가치 m이 10000원을 넘길수 없다는 점에서 착안해서 10001로 설정한다. 0원은 아무 화폐도 사용하지 않으면 그 자체로 달성이 가능하기 때문에 값을 0으로 처리한다.

| 인덱스 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 값 | 0 | 10001 | 10001 | 10001 | 10001 | 10001 | 10001 | 10001 |

이제 작은 화폐 단위부터 순서대로 반복해서 확인한다. 우선 2원 부터 시작하면, 2원을 만들기 위해서는 2원짜리 하나면되고 4원은 2원짜리 두개, 6원은 2원짜리 세개이기 때문에 아래와 같이 리스트가 갱신된다.

| 인덱스 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 값 | 0 | 10001 | 1 | 10001 | 2 | 10001 | 3 | 10001 |

두번째 화폐단위인 3을 확인하면, 3원을 만들기위해 3원짜리 하나가 필요하고 5원을 만들기 위해서는 2원과 3원 하나씩 필요하고 6원을 만들기 위해서는 3원짜리 두개가 필요하고 7원을 만들기 위해서는 2원짜리 두개와 3원짜리 하나가 필요하다.

| 인덱스 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 값 | 0 | 10001 | 1 | 1 | 2 | 2 | 2 | 3 |

마지막으로 5원도 적용하면 최종적으로 아래와같은 리스트가 만들어진다.

| 인덱스 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 값 | 0 | 10001 | 1 | 1 | 2 | 1 | 2 | 2 |

여기서 생각해보면, 7원을 만드는경우 5원을 적용하기 전에는 3인데 5원을 사용하면 2가된다. 이는 d[7]에서 5만큼 앞에 있는 인덱스인 2의 값인 1에다가 5원 한번만 더해주면 이전값과 비교해서 더 작은 값을 사용하기 때문에 이를 활용하면 점화식을 만들 수 있다. 그렇게 만들어진 최종 코드는 아래와 같다.

```python
"""
입력조건 : 첫째 줄에 n, m. 이후 n개의 줄에는 각 화폐의 가치 입력(단, 10000원 이하)
출력조건 : 첫째줄에 m원을 만들기 위한 최소한의 화폐개수 출력. 불가능할 땐 -1 출력
"""

n, m = map(int, input().split())
arr = []
for i in range(n):
	arr.append(int(input()))

d = [10001] * (m+1)

d[0] = 0
for i in range(n):
	for j in range(arr[i], m+1):
		if d[j-arr[i] != 10001: # i-k원을 만드는 방법이 존재하는 경우
			d[j] = min(d[j], d[j - arr[i]] + 1)

if d[m] == 10001:
	print(-1)
else:
	print(d[m])
```
