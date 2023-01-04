---
title: Kubernetes - Deployment
categories: [Kubernetes, Notion]
---

# Deployment란?

![k8s-deployment0](/images/k8s-deployment0.png)

쿠버네티스 클러스터를 운영할 때 replicaset이나 pod을 단독으로 사용하는 경우는 거의 없고, 대부분의 경우 replicaset과 pod을 deployment라는 오브젝트를 yaml파일로 정의하는 방식으로 사용합니다. 실제로 Deployment는 쿠버네티스에서 가장 널리 사용되는 오브젝트 중 하나이며, 쿠버네티스의 상위 수준의 리소스에 해당합니다.

Deployment는 replicaset과 pod의 선언적인 업데이트를 제공합니다. 이는 Deployment로 원하는 상태(Desired state)를 정의하고 현재의 상태에서 원하는 상태로의 변화를 만드는 것으로 묘사됩니다. 이뿐만아니라 업데이트 이력을 관리하여 롤백(Rollback)하거나 특정 버전으로 돌아가는 것도 가능합니다.

# deployment 간단 실습

이전에 정의한 replicaset를 deployment로 만들어 간단한 실습을 진행해보겠습니다.

```python
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```

사실 kind가 replicaset에서 deployment로만 바뀌면 됩니다. 다른 것들은 완전히 동일합니다.

![k8s-deployment1](/images/k8s-deployment1.png)

결과를 조회해봐도 replicaset과 거의 동일하지만, deployment가 추가로 생성됐다는 차이가 있습니다. replicaset하고 이렇게 비슷해보이는데 왜 deploymet를 사용하느냐라는 의문이 생길 수 있습니다. 그러나 deployment의 진가는 pod를 새로운 이미지로 업데이트할 때 나타납니다.

```python
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v2
```

이미지를 v1에서 v2로 업데이트하고 이를 다시 적용시킨뒤 확인해보면 아래와 같은 결과를 얻을 수 있습니다.

![k8s-deployment2](/images/k8s-deployment2.png)

deployment는 pod을 새로운 이미지로 업데이트 할때 replicaset를 이용합니다. 새로운 replicaset을 생성하고 해당 replicaset이 새로운 버전의 pod를 생성하는 프로세스입니다. 물론 엄밀히 말하면 pod을 새로운 버전으로 업데이트 한다는 것은 옳지 않은 표현입니다. 새로운 버전의 pod을 생성하고 기존의 pod을 제거한다는 것이 정확한 표현입니다. 그럼에도 결과적으로 pod이 업데이트된 효과를 얻는다는 것에는 변함이 없습니다.

`k describe deploy echo-deploy` 커맨드로 자세한 동작과정을 살펴보면 아래와 같습니다.

![k8s-deployment3](/images/k8s-deployment3.png)

새로운 replicaset을 생성하고 관리하는 pod의 개수를 0 → 1로 조정하고 정상적으로 pod이 동작하면 기존 replicaset을 4 → 3으로 조정합니다. 그리고 새로운 replicaset을 1 → 2로 조정하고 정상적으로 pod이 작동하면 기존 replicaset을 3 → 2로 조정합니다. 이를 반복하여 새로운 replicaset이 4개로 조정되고 기존 replicaset이 0으로 되면 업데이트가 완료된 것입니다.

이들을 Desired와 Current의 조정과정으로 표현하는데, 이미지 업데이트 적용 이후 기존의 replicaset의 Desired와 current가 순차적으로 조정되면서 0이되고 새로운 replicaset의 Desired와 Current도 순차적으로 조정되면서 4로 일치하면 업데이트 작업이 완료되는 개념입니다.

# 컨트롤러 작동 방식

![k8s-deployment4](/images/k8s-deployment4.png)

1. `Deployment Controller`는 Deployment조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
2. `Deployment Controller`가 원하는 상태가 되도록 `ReplicaSet` 설정
3. `ReplicaSet Controller`는 ReplicaSet조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
4. `ReplicaSet Controller`가 원하는 상태가 되도록 `Pod`을 생성하거나 제거
5. `Scheduler`는 API서버를 감시하면서 할당되지 않은 `Pod`이 있는지 체크

    unassigned

6. `Scheduler`는 할당되지 않은 새로운 `Pod`을 감지하고 적절한 `노드`에 배치

    node

7. 이후 노드는 기존대로 동작

이전 replicaset이 동작하던 방식에서 deployment를 관리하는 deployment controller가 추가된 형태입니다.

# 버전 관리

deployment는 업데이트 이력 즉, 변경 상태를 기록합니다. 이는 `kubectl rollout history` 커맨드로 확인할 수 있고, 특정 revision을 옵션으로 지정해주면 해당 revision의 히스토리 상세를 확인할 수도 있습니다.

![k8s-deployment5](/images/k8s-deployment5.png)

이를 활용하여 이전 버전으로 롤백하거나 특정 버전을 지정하여 롤백할수도 있습니다. 그리고 `--to-revision=2`와 같이 특정 revision을 지정하여 롤백하는 것도 가능합니다.

![k8s-deployment6](/images/k8s-deployment6.png)

특징적인 것은 롤백 후 revision이 3으로 갱신되는 부분입니다.

앞서 한 내용을 생각해보면 버전 1에서 2로갔다가 다시 1로 돌아왔습니다. 그런데 revision에는 두개밖에 나와있지않은 것을 볼 수 있습니다.

이 이유는 쿠버네티스는 이미지의 버전이 동일하면 더 이전의 것을 삭제하도록 되어있기 때문입니다. 그래서 revision이 3개가 아니라 2개만 존재하고 revsion 번호도 2,3으로 되어있는 것입니다.

# 배포 전략

deployment에는 다양한 배포 전략이 존재합니다. 대표적으로 recreate와 rollingupdate가 있고 기본값은 롤링업데이트(Rolling-Update) 방식입니다. 이 단락에서는 rollingupdate 방식을 사용하여 동시에 업데이트하는 pod의 개수를 변경하는 간단한 실습을 진행해보겠습니다.

```python
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy-st
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 3
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
          livenessProbe:
            httpGet:
              path: /
              port: 3000
```

우선 deployment를 적용시키고 앞선 단락에서처럼 이미지를 v2로 업데이트 시켜보겠습니다. 이번엔 `kubectl set image deploy echo-deploy-st echo=ghcr.io/subicura/echo:v2`커맨드로 바로 이미지 업데이트를 진행합니다.

![k8s-deployment7](/images/k8s-deployment7.png)

해당 deployment의 상세 이벤트를 확인해보면 이전에 하나씩 pod을 생성하지않고 한번에 3개씩 생성하는 것을 확인할 수 있습니다. 이는 롤링업데이트 옵션으로 `maxSurge=3`과 `maxUnavailable=3`을 주었기 때문입니다.

## 최대 불가(Max Unavailable)

이 옵션은 업데이트 프로세스 중에 사용할 수 없는 최대 pod의 수를 지정하는 필드입니다. 선택 옵션이며 숫자로 지정해줄 수도있고 원하는 파드 비율(ex. 10%)가 될 수도 있습니다.

가령 이 값을 30%로 설정했다고 하면 롤링업데이트 시작 시 즉각 이전 replicaset의 크기를 의도한 pod 중 70%를 scale down 할수 있습니다. 새 pod가 준비되면 기존 replicaset을 scale down할 수 있으며 업데이트 중에 항상 사용 가능한 전체 pod의 수는 의도한 pod의 수의 70%이상이 되도록 새 replicaset을 scale up 할수 있습니다.

기본값은 25%입니다.

## 최대 서지(Max Surge)

이 옵션은 의도한 pod의 수에 대해 생성할 수 잇는 최대 pod의 수를 지정하는 필드입니다. 선택 옵션이며 이 값도 역시 숫자나 비율로 지정할 수 있습니다.

가령 이 값을 30%로 설정했다고 하면 롤링업데이트 시작 시 새 replicaset의 크기를 즉시 조정해서 기존 및 새 pod의 전체 개수를 의도한 pod의 130%를 넘지 않도록 합니다. 기존 pod가 죽으면 새로운 replicaset은 scale up 할 수 있고, 업데이트 하는 동안 항상 실행하는 총 pod의 수는 최대 의도한 pod의 수의 130%가 되도록 보장합니다.

이 역시 기본값은 25% 입니다.

# 마무리

deployment는 맨 첫단락에서 언급했듯이 쿠버네티스 운영환경에서 가장 많이 사용되는 오브젝트 중 하나입니다. 이외에도 StatefulSet, DaemonSet, CronJob 등이 있지만 사용법에 있어 큰 차이가 있지는 않습니다.
