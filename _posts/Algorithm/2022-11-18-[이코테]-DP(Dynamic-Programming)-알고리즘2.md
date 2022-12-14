---
title: 이코테 Ch.6 DP(Dynamic Programming) 알고리즘 (2)
categories: [Algorithm, Dynamic Programming]
---
# 1로 만들기

정수 X가 주어질 때 정수 X에 사용할 수 있는 연산은 4가지

- X가 5로 나누어떨어지면 5로 나누기
- X가 3으로 나누어떨어지면 3으로 나누기
- X가 2로 나누어 떨어지면 2로 나누기
- X에서 1빼기

정수 X가 주어질때 1로만드는 최소 연산 횟수 구하기

```python
n = int(input())

d = [0] * 100001

for i in range(2, n+1):
	d[i] = d[i-1] + 1
	if i % 5 == 0:
		d[i] = min(d[i], d[i//5] +1)
	if i % 3 == 0:
		d[i] = min(d[i], d[i//3] +1)
	if i % 2 == 0:
		d[i] = min(d[i], d[i//2] +1)

print(d[n])
```

우선 정수 i가 들어오면 1을 빼는 연산을 수행하고 카운트를 1 올린다. d[2]부터 순차적으로 채워나가는 과정이기 때문에 d[i-1]에서 1을 더해주는 방식을 사용한다. 이후 5,3,2로 나누어떨어지는 확인하여 i번째 기입된 카운트와 i를 5,3,2로나눈 몫의 값에서 1더한 카운트와 비교해서 더 작은 값을 기입한다.

# 개미 전사

여러개의 식량창고는 일직선으로 이어진 형태로 각 식량창고에는 정해진 수의 식량을 저장하고 있고 개미 전사는 선택적으로 식량창고를 약탈한다. 단, 서로 인접한 곳을 약탈하면 메뚜기 정찰병이 눈치를 챈다.

식량창고의 식량이 주어질때, 약탈할 수 있는 식량의 최대값 구하기

```python
n = int(input())

arr = list(map(int, input().split())

d = [0] * 101

d[0] = arr[0]
d[1] = max(arr[0], arr[1])
for i in range(2, n):
	d[i] = max(d[i-1], d[i-2] + arr[i])

print(d[n-1])

```

식량 창고의 왼쪽부터 차례대로 식량창고를 털지 말지 결정해야한다. i번째의 식량 창고를 털지말지 결정해야하는 순간이 오면, i-1까지 의 식량창고에서 약탈가능한 식량의 수와 i-2까지의 식량창고에서 약탈가능한 식량창고와 i번째의 식량창고에 있는 식량의 양을 더한 값을 비교해 더 큰 값을 선택한다.

# 정리

두 문제 모두 어떤 것이 DP테이블에 들어가 있는지를 생각하는게 중요하다. 바텀업 방식이기 때문에 반복문이 순차적으로 돌아가면 DP테이블의 결과값을 집어넣기 때문에 이후의 작업에서 이를 어떻게  가져와서 계산할 수 있을지를 생각해봐야한다. 즉, 결국 이를 통해 점화식을 얼마나 잘 떠올리냐에 문제 해결의 50%가 달려있다.
