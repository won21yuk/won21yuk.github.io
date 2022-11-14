---
title: 이코테 Ch.5 이진탐색(Binary search) 알고리즘 (2)
categories: [Algorithm, Binary search]
---

# 부품찾기

```python
"""
부품 n개가 있고 각 부품은 고유번호가 있음
m개의 부품을 구매하려는 고객에게 견적서를 만들어야함
부품이 있으면 yes, 없으면 no 출력
"""

def binary_search(arr, target, start, end):
  while start <= end:
    mid = (start + end) // 2
    if arr[mid] == target:
      return mid
    elif arr[mid] > target:
      end = mid - 1
    else:
      start = mid + 1
  return None

n, m = map(int,input().split())
arr = list(map(int, input().split()))
arr.sort() # 이진탐색을 위한 정렬
arr_req = list(map(int, input().split()))

for i in arr_req:
  result = binary_search(arr, i, 0, n-1)
  if result != None:
    print('yes', end=' ')
  else:
    print('no', end=' ')
```
항상 기억해하는건 배열을 받을 때 이진탐색을 위한 정렬을 반드시 실행해야한다는 점이다.

따라서 n개의 부품을 나열하고 그 다음 m개의 부품을 확인하기 위한 이진탐색을 수행한다.

이렇게 되면 최악의 경우를 산정해도 시간복잡도는 O((M+N)*logN)이다.

## 계수정렬로 풀기
이문제는 계수 정렬로도 풀수가 있다.

특정 번호의 부품이 있는지만 확인하면 되기때문에 계수정렬을 한 후, 특정 부품 번호가 1인지 아닌지만 판별하면 되기 때문이다.

```python
n = int(input())
arr = [0] * 1000001

for i in input().split():
  arr[int(i)] = 1

m = int(input())
arr_req = list(map(int, input().split()))

for i in arr_req:
  if arr[i] == 1:
    print('yes', end=' ')
  else:
    print('no', end=' ')
```

## 집합 자료형으로 풀기
이문제는 단순 집합 자료형을 통해서도 풀수 있다. 단순히 특정 수가 한번이라도 등장했는지만 검사하면 되기 때문이다.

```python
n = int(input())
# 모든 부품번호를 입력받아 집합(set) 자료형에 기록
arr = set(map(int, input().split()))

m = int(input())
arr_req = list(map(int, input().split()))

for i in arr_req:
  if i in arr:
    print('yes', end=' ')
  else:
    print('no', end=' ')
```

이러한 형태의 문제 풀이는 특정 데이터가 존재하는 지를 검사할 때 가장 간결하게 코드를 작성할 수 있어 효과적이다.

# 떡볶이 떡 만들기
이 문제는 전형적인 이진탐색 문제이면서, 파라메트릭 서치 유형의 문제이다. 파라메트릭 서치는 최적화 문제를 결정문제로 바꾸어 해결하는 기법이다. 원하는 조건을 만족하는 가장 알맞은 값을 찾아주는 문제에 주로 사용된다.

가령 범위 내 조건을 만족하는 가장 큰 값을 찾아야하는 최적화 문제의 경우, 이진탐색으로 결정 문제를 해결하면서 범위를 좁혀갈 수 있다. 이때문에 코딩 테스트에서는 파라메트릭 서치 문제를 이진탐색으로 해결한다.

이 문제 풀이는 적절한 높이를 찾을 때까지 절단기의 높이 h를 반복해서 조정하면 된다.

```python
"""
절단기 높이 h를 지정하면 줄지어진 떡을 한번에 절단
손님이 요청한 길이가 m일때 적어도 m만큼의 떡을 얻기위해 절단기에 설정할 수 있는 높이의 최댓값 구하기
"""

n, m = list(map(int, input().split()))
arr = list(map(int, input().split()))

start = 0
end = max(arr)

result = 0
while(start <= end):
  total = 0
  mid = (start+end)//2
  for x in arr:
    # 잘랐을 때의 떡의 양 계산
    if x > mid:
      total += x - mid
  # 떡의 양이 부족한 경우 더 많이 자르기
  if total < m:
    end = mid - 1
  # 떡의 양이 충분한 경우 덜 자르기
  else:
    result = mid
    start = mid + 1

print(result)
```
