---
title: Kubernetes - Replicaset
categories: [Kubernetes, Notion]
---

# Replicaset이란?

![k8s-rs0](/images/k8s-rs0.png)

Pod는 쿠버네티스의 기본 단위로써 여러개의 컨테이너를 추상화하여 하나의 어플리케이션으로 동작하게 만드는 컨테이너의 묶음입니다. 만약 yaml파일에 pod만 정의해서 생성할 경우, 이 pod의 Lifecycle은 컨테이너 삭제시 영원히 사라지게 됩니다. 그래서 yaml파일에 pod만 정의해서 생성하면 해당 pod는 오직 쿠버네티스의 사용자인 본인에 의해 관리됩니다.

그러나 실제로 외부 사용자의 요청을 처리해야하는 마이크로 서비스 구조의 pod라면 이러한 방식을 사용하기 어렵습니다. 쿠버네티스의 기본 단위가 pod이기 때문에 여러개의 pod를 생성해 외부 요청을 각 pod에 분배하는 방식을 사용해야하기 때문입니다.

여러개의 동일한 pod를 직접 생성하는 방법은 여러가지 이유로 적절하지 않습니다. 우선 동일한 pod의 개수가 많아질수록 일일이 정의하는 것은 그 자체로 매우 비효율적인 일이며, pod가 어떠한 이유로든지 삭제되거나 pod가 위치한 워커노드에 장애가 발생해 더 이상 pod에 접근할 수 없을 때 직접 pod를 삭제하고 다시 생성하지 않는 한 해당 pod는 다시 복구되지 않습니다.

따라서 이러한 경우 replicaset이라는 쿠버네티스 오브젝트를 함께 사용하는 것이 일반적입니다. replicaset의 목적은 replica pod 집합의 실행을 항상 안정적으로 유지하는 것에 있습니다. 워커노드 등의 장애로 pod에 접근할 수 없더라도 다른 워커노드에서 pod를 다시 생성하여 replicaset에 정의된 pod의 수를 유지합니다.

쿠버네티스의 초창기에는 pod를 복제하고 pod가 실패했을 때 새 pod를 만드는 레플리케이션 컨트롤러가 유일했습니다. 그러나 레플리케이션 컨트롤러의 문제점 및 기능 개선을 위해서 replicaset이라는 컨트롤러가 새로이 추가되었고 현재는 replicaset 또는 deployment를 사용하는 것을 권장하고 있습니다.

# replicaset 간단 실습

이전 pod 포스팅에서 만들었던 pod을 replicaset으로 만들어보는 실습을 진행해 보겠습니다.

```python
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-rs
spec:
  replicas: 1
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

pod를 정의할 때보다는 다소 복잡해졌습니다. kind가 replicaset으로 정의되고 spec에 replicas를 정의해주며, selector를 통해 pod의 label을 체크하도록 합니다. 그리고 template를 통해서는 생성할 pod의 metadata 즉, label를 정의해 줄 수 있습니다.

![k8s-rs1](/images/k8s-rs1.png)

작성한 yaml 파일을 실행시킨후 확인해보면 replicaset과 pod가 생성된 것을 확인할 수 있습니다. replicaset은 앞서 정의한 pod의 label를 바라보고 있다가 일치하는 label의 pod이 없다면 새로운 pod을 생성합니다. `kubectl get pod —show-labels`를 통해 pod의 label을 확인하면 앞선 yaml파일에서 metadata로 정의한 label(`app=echo`, `tier=app`)로 pod이 생성된 것을 확인할 수 있습니다.

![k8s-rs2](/images/k8s-rs2.png)

그럼 임의로 label를 제거해보면 어떻게 될까요?

![k8s-rs3](/images/k8s-rs3.png)

생성된 pod의 app label을 제거한 후 다시 pod을 조회해보면 `app=echo`이며 `tier=app`을 만족하는 새로운 pod가 만들어진 것을 확인할 수 있습니다.

![k8s-rs4](/images/k8s-rs4.png)

그럼 app label을 다시 추가해보면 어떻게 될까요?

![k8s-rs5](/images/k8s-rs5.png)

replicas에 정의된대로 pod의 개수를 1개로 유지하기 위해 기존의 pod을 제거하고 혼자만 남게 됩니다.

# replicaset의 동작방식

![k8s-rs6](/images/k8s-rs6.png)

1. `ReplicaSet Controller`는 ReplicaSet조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
2. `ReplicaSet Controller`가 원하는 상태가 되도록 `Pod`을 생성하거나 제거
3. `Scheduler`는 API서버를 감시하면서 할당되지 않은 `Pod`이 있는지 체크

    unassigned

4. `Scheduler`는 할당되지 않은 새로운 `Pod`을 감지하고 적절한 `노드`에 배치

    node

5. 이후 노드는 기존대로 동작

이전 pod의 생성과정에 replicaset controller가 추가되어 replicaset을 관리하는 형태가 됩니다. 여전히 pod의 할당은 scheduler가 전담하고 kubelet, container도 각자 맡은 역할을 그대로 수행합니다.

# Scale out

replicaset을 사용하면 손쉽게 pod을 여러개로 복제할 수 있습니다. 앞서 작성한 yaml파일에서 replicas를 4로 변경한 후 작동시켜보면 아래와 같이 4개의 pod이 생성되는 것을 확인할 수 있습니다.   이는 기존에 생성한 pod에 3개의 pod이 추가된 것입니다.

![k8s-rs7](/images/k8s-rs7.png)

# 마무리

결국 replicaset은 label을 통해 pod을 계속 모니터링하여 replicas의 개수만큼의 동일한 pod을 유지하는 역할을 수행합니다. label을 이용하여 pod을 체크한다는 것이 핵심적인 특징이며, 이때문에 pod의 label이 겹치지 않도록 신경써서 정의해야합니다.

pod처럼 단독으로 replicaset을 사용하는 경우는 거의 없고 다음 포스팅에서 다룰 deployment를 통해 replicaset을 사용하는 것이 대부분입니다.
