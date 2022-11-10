---
title: 문법정리 2. return None
categories: [ETC, 문법정리]
---
알고리즘을 문제를 풀다가 마주한 return None.

예시 코드에서 함수를 선언하고 return 값으로 None을 주길래 당연히 None값만 반환될 것이라 생각했다. 그런데 실제 동작은 그렇지 않았다.

return None이 있던 없던 함수 자체가 작동하는데는 별 영향을 미치지 않는다.

그럼 return None이 있던 없던 함수는 멀쩡히 동작하던데 왜 return None을 쓰는것이고 어떤 역할을 맡고있는 것인지를 살펴보려한다.

# TEST 1: 일반 함수에서 return None

```python
def return_test(n):
	if n == 1:
		n += 1
  return None
t = return_test(5)

print(t)
# None
```

난 위의 함수를 실행하면 무조건 t값은 None이 될거라 생각했다. 그리고 실제로 t는 None으로 출력한다.

그럼 조건문이 참이 되도록 하면 어떨까?

난 이번에도 당연히 None으로 나올거라고 생각했다. 내가 아는 return은 함수의 결과로서 반환되는 값을 적는 것이라고 알고 있기 때문이다.

```python
def return_test(n):
	if n == 1:
		n += 1
  return None

t = return_test(1)

print(t)
# None
```

실제로 None이 출력한다. 안의 조건문이 참이던 거짓이든 상관이 없는 듯하다.

# TEST 2 : While 문 사용

그럼 내가 봤던 예제처럼 while문을 사용해보면 어떨까?

```python
def return_test(n):
	while n >= 1:
		if n == 10:
			break
		else:
			n += 1

	return None

t = return_test(2)
print(t)
# None
```

이것도 None이 출력된다. 그럼 break를 return으로 주면 어떨까?

```python
def return_test(n):
	while n >= 1:
		if n == 10:
			return n
		else:
			n += 1

	return None

t = return_test(2)
print(t)
# 10
```

이젠 10이 출력된다. 드디어 답을 찾았다.

안의 반복문이 정상적으로 작동하고 종료조건이면 리턴값을 반환하도록하면 None이 아닌 종료조건의 리턴값인 n이 출력된다.

while문이 작동하지 못하도록 하기 위해서 n에 -1이라는 값을 주면 어떨까? 아마 None이 출력되지않을까 예상해본다.

```python
def return_test(n):
	while n >= 1:
		if n == 10:
			return n
		else:
			n += 1

	return None

t = return_test(-1)
print(t)
# None
```

정확이 None이 출력된다. 그럼 한가지가 더 궁금해진다. while문을 안쓰고 조건문만 쓰면 똑같이 작동할까?

```python
def return_test(n):
	if n == 10:
		return n
	else:
		n += 1
	return None

t1 = return_test(10)
print(t1)
# 10

t2 = return_test(-1)
print(t2)
# None
```

역시 조건문을 충족하면 해당 return값이 return None보다 우선된다는 결과를 확인했다.

# 왜 return None을 써야하는가?

return None이 작동하는 방식은 이해를 했다.

그런데 왜 return None을 써야할까?

```python
def test1():
	a = 1

def test2():
	return

def test3():
	return None

print(test1(), test2(), test3())
# None, None, None
```

위에 코드에서 보다시피 return을 아예 안써도, 아니면 return 만 쓰고 값을 따로 안써도, 그리고 리턴값으로 None을 쓰던 모두 결과는 None을 반환한다.

근데 굳이 None을 쓴다는 것은 무언가 목적성이 있는 것같이 느껴진다.

그래서 관련 내용을 찾아봤다.

## CASE 1: 명시적으로 구분해줘야 하는 경우

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

위의 코드는 내가 return None에 대해 궁금해지게 만든 이진 탐색을 함수로 구현한 코드이다.

이 코드의 맨 아래를 보면, None이면 ‘원소가 존재하지 않습니다’를 출력하고 아니면 result +1을 출력하도록 했다.

이런식으로 함수의 결과가 None인지 아닌지에 따라 다르게 작동하도록 하려는 코드를 작성해야하는 경우에는 return 값을 None으로 명시해서 코드의 가시성을 높히는 것이 좋다.

## CASE 2 : early return을 사용하는 경우

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

```

앞서 사용했던 이진 탐색 코드를 다시한번 가져왔다.

이 코드에서 if에서 바로 return을 걸어주는게 바로 early return이다.

근데 여기는 구체적인 값을 넣어줘서 mid 값이 리턴되도록 해주었지만, 이 동작을 생각해보면 early return이 무엇인지 감이온다.

while로 loop를 걸고 if를 충족하면 return값을 반환하면서 바로 종료가 된다. break와 거의 같은 역할을 수행한다는 것이다. 실제로 위의 함수도 작동을 시켜보면 array[mid] == target이 되버리면 elif나 else의 코드가 필요없다.  그대로 loop가 끝나버리니까.

따라서 return만 단독으로 써서 그냥 break와 완전히 동일한 효과를 보도록 할수도 있다.

```python
def find_prisoner_with_knife(prisoners):
    for prisoner in prisoners:
        if "knife" in prisoner.items:
            prisoner.move_to_inquisition()
            return # no need to check rest of the prisoners nor raise an alert
    raise_alert()
```

위의 코드가 바로 break 대용으로 early return을 사용한 예이다. 이 경우에는 반복문을 종료시키는데 return의 의미가 있을 뿐이기에 None을 사용할 필요가없다. None은 앞선 CASE1처럼 따로 사용될 것이 아니기 때문이다.

## CASE 3 : 단순 연산만이 목적인 경우

```python
def set_some_fruit(fruit):
    fruits = []
    if is_fruit(fruit):
        fruits.append(fruit)
```

위 코드는 과일인지 판별 후 과일이면 배열에 넣는 코드이다.

단순한 함수의 작동을 위한 조건문이기 때문에 return을 줄 필요가없다.

또한 반복문이 아닌 단순 판별을 위해 조건문이 쓰였기때문에 early return도 필요가 없다.

## 결론

사실 위의 케이스들이 진리는 아니다. 참고하면 좋다 정도이지 사실 무조건 이 규칙에 따를 필요는 없다.

다만 함수에 early return과 함께 사용되서 함수값이 None인지 아닌지에 따라 다른 코드를 실행시키고자 한다면 이 내용이 읽기 좋은 코드를 만드는데 도움이 될 듯하다.
