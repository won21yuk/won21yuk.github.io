---
title: 이코테 Ch.2 구현 알고리즘 (2)
categories: [Algorithm, Implementation]
---

좌표평면에서의 이동에 대한 알고리즘은 각 커멘드에 따른 이동방향을 확실히 지정해줘야한다.

앞선 구현 알고리즘(1)에서는 L, R, U, D라는 이동방향마다 X,Y값들의 이동을 dx, dy라는 리스트에 지정해 주었다.

이번 왕실의 나이트 문제도 이와 비슷한 방식이다.

```python
"""
8 X 8 체스판
L자 형태로 이동
나이트위 위치는 input으로 받음
열은 알파벳 a~h
행은 숫자 1~8
"""
input_data = input()
row = int(input_data[1])
# ord 함수 : 문자열을 인자로 받아 유니코드로 반환시켜주는 함수. (chr 함수와 정반대의 역할)
# 1을 더해주는건 column 값이 0이 되는 것을 막기 위함.
column = int(ord(input_data[0])) - int(ord('a')) + 1

# 나이트가 이동가능한 방향 정의
steps = [(-2,-1), (-1,-2), (1,-2), (2, -1), (2,1), (1,2), (-1,2), (-2,1)]

result = 0
for step in steps:
  next_row = row + step[0]
  next_column = column + step[1]
  # 체스판 위의 좌표값이라면, 카운트 증가
  if next_row >= 1 and next_row <= 8 and next_column >= 1 and next_column <= 8:
    result += 1

print(result)




```
