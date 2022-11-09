---
title: 문법정리 1. python에서의 continue와 pass
categories: [ETC, 문법정리]
---

알고리즘을 공부하다보니 반복문에서 예외를 판별하는 조건문을 적용할 때, continue가 사용되는 것을 목격했다.

나는 보통 이런 경우에는 pass만 사용했는데, 특별히 continue가 필요한 상황이 있는지에 대해 궁금증이 생겼다.

이를 위해 continue와 pass에 대해 어느 정도 정리할 필요성 있어 관련 내용을 포스팅하려 한다.

## pass
이걸 작성하면서 생각해보니 나는 반복문이던 try ~ except문이던 예외처리할 때는 모든 경우 pass를 사용했던것 같다.

몇 개 보고 가자면,
- try ~ except
```python
...
if df.loc[i, '위도'] == 0:
    try:
        x = json_obj['documents'][0]['x']
        y = json_obj['documents'][0]['y']
        df.loc[i, '경도'] = x
        df.loc[i, '위도'] = y
    except:
        pass
...
```

- for 문
```python
...
for k in range(1, npp + 1):
    if k % 3 == 0:
        pass
    else:
        dic = dict()
        corp_nn = driver.find_element(by=By.CSS_SELECTOR, value='#contents > div.contents > section.spacial-page.detail-intro > div > div.infomation > p.k-name').text
        category = driver.find_element(by=By.CSS_SELECTOR, value='#contents > div.contents > section.spacial-page.detail-intro > div > div.infomation > div > dl > dd:nth-child(2)').text
...
```
다 이런식으로 코드를 구성했었다.

단순히 조건에 맞으면, 아무것도 하지말고 다음 루프로 넘어가도록 하기위한 의도를 가지고 작성했고 실제로 내가 생각한대로 잘 작동했었다.

이는 pass가 실행은 되나 아무 동작을 하지않는 코드이기 때문인데, 이를 좀 더 간단한 코드로 표현해 보자.
```python
for i in range(1, 6):
    if i%2 == 0:
        pass
        print(f'{i}는 짝수입니다!')
        pass
    else:
        print(f'{i}는 짝수가 아닙니다..')

'''
[결과]
1는 짝수가 아닙니다..
2는 짝수입니다!
3는 짝수가 아닙니다..
4는 짝수입니다!
5는 짝수가 아닙니다..
'''
```
위의 함수는 반복문 안에서 짝수 여부를 구분 한다.

위의 함수의 결과는 짝수면 '짝수입니다!'가 출력되고 아니면 '짝수가 아닙니다..'가 출력되는 것이다.

이는 조건문 안에서 pass -> print -> pass 의 순서대로 작동한다는 것을 의미한다.

이처럼 if문 안의 pass는 실행되지만 아무일도 일어나지않고 다음 줄의 코드가 실행된다. 즉, 소스코드가 명목적으로 존재할 뿐이라는 것이다.

이때문에 try ~ except에서 예외상황 발생시 그냥 바로 넘겨버리거나, 반복문에서 if문을 사용해 특정 조건을 걸러내고 진짜 중요한 코드는 else에서 작동하게끔하기 위한 명목적인 코드 작성 시, 이를 받아주기 위해 사용하는 것이 일반적이다.

## continue
같은 함수를 사용할 때, pass를 continue로만 바꾸면 어떻게 될까?
```python
for i in range(1, 6):
    if i%2 == 0:
        continue
        print(f'{i}는 짝수입니다!')
        pass
    else:
        print(f'{i}는 짝수가 아닙니다..')

'''
[결과]
1는 짝수가 아닙니다..
3는 짝수가 아닙니다..
5는 짝수가 아닙니다..
'''
```
print에 앞서서 continue를 사용하니 continue 뒤에 존재하는 출력문이 작동하지 않는다.

따라서 이 함수는 짝수가 아닐 경우만 출력문이 작동하게 된다.

이는 continue는 실행되는 즉시 다음 순번의 loop를 실행하기 때문인데, 이 때문에 continue 이후의 코드는 실행되지 않게 된다.

위 코드를 앞선 pass문과 같은 결과가 나오도록 하고, 동시에 continue를 쓰고싶다면, 아래와 같이 작성하면 된다.
```python
for i in range(1, 6):
    if i%2 == 0:
        pass
        print(f'{i}는 짝수입니다!')
        continue
    else:
        print(f'{i}는 짝수가 아닙니다..')

'''
[결과]
1는 짝수가 아닙니다..
2는 짝수입니다!
3는 짝수가 아닙니다..
4는 짝수입니다!
5는 짝수가 아닙니다..
'''
```
pass는 실행되지만 아무 작동도 하지않고 다음 코드를 실행시키고 continue는 바로 다음 loop를 실행시키도록 만들기 때문이다.

## 요약
- 둘다 반복문에서 조건문으로 예외 처리 할때 주로 사용됨
- pass : 실행은되나 어떤 작동이 일어나지는 않는 명령어
  - 명목적인 소스코드를 작성시 사용
  - 뒤 이은 코드들 실행됨
- continue : 다음 loop를 바로 작동시키는 명령어
  - 바로 다음 loop를 실행시키고자 할 때 사용
  - 뒤 이은 코드들 실행 안됨
