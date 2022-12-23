---
title: Kubernetes Practice - minikube 설치 및 테스트
categories: [Kubernetes, Practice]
---

쿠버네티스를 운영환경에 설치하려면 최소 3대의 마스터 서버와 컨테이너 배포를 위한 n개의 워커서버가 필요하다. 이러한 구성 작업을 직접 하는 것은 꽤나 복잡하고 만약 퍼블릭 클라우드 서비스를 이용한다고 해도 각 서비스마다 사용법이 다 다르기 때문에 첫 학습을 시작하는 시점에서 이를 해내기는 어렵다. 따라서 간단하게 단일노드에 마스터와 워커를 띄워서 실습환경을 구축하도록 한다.

쿠버네티스를 실습/개발 환경으로 구축하는 데 사용되는 도구들은 여러가지가 있다. 대표적으로 minikube, k3s, docker for desktop, kind 등이 있다. 이들은 쿠버네티스를 기반으로 하되 경량화한 버전이라고 보면 된다. 이 중 minikube가 대부분의 환경에서 사용할 수 있고 간편하며 무료이기 때문에 가장 많이 사용되고 따라서 앞으로의 쿠버네티스 실습도 minikube를 사용한다.

# minikube 설치

쿠버네티스 클러스터를 실행하기 위해선 최소한 scheduler, controller, api-server, etcd, kubelet, kube-proxy를 설치해야한다. minikube는 이런것들을 쉽게 구성할 수 있도록 해준다. 또한 minikube는 windows, macos, linux 등의 os에서 모두 사용가능하고 다양한 가상환경(Docker, Virtualbox, KVM..)을 지원한다.

![kube-practice-minikube0](/images/kube-practice-minikube0.png)

홈페이지에는 minkube를 사용하기 위한 최소한의 스펙이 명시되어 있다. 만약 가상머신에서 minikube를 사용한다면 이를 고려해야한다.

![kube-practice-minikube1](/images/kube-practice-minikube1.png)

내가 사용할 gcp 인스턴스는 이전에 docker 실습을 진행했었기 때문에 이를 활용해 docker 기반으로 minikube를 사용할 계획이다. 따라서 아래에 적힌 minikube 설치 과정은 docker가 기본적으로 설치되어 있다는 가정하에서의 내용이다.

```python
# kubectl 설치(kubernetes용 CLI 도구)
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl && chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

# 설치확인
kubectl version

# minikube 설치
sudo curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 도커로 minikube 실행
# 도커가 실행된 상태여야지만 vm으로 docker를 지정하여 minikube실행 가능
sudo systemctl start docker
minikube start --vm-driver=docker
```

![kube-practice-minikube2](/images/kube-practice-minikube2.png)

minikube 관련한 커맨드는 아래와 같다.

```python
# minikube 상태
minikube status

# minikube ip 확인(이후 접속 테스트에 필요)
minikube ip

# minikube 종료
minikube stop

# minikube 삭제
minikube delete
```

# 도커 컴포즈 vs 쿠버네티스

minikube를 설치하였으니 간단한 테스트를 진행해본다. 이를 위해 PHP와 MySQL로 구성된 워드프레스를 도커 컴포즈와 쿠버네티스로 배포해보고 이들의 차이점을 간단히 알아본다.

## 도커컴포즈로 워드프레스 배포

![kube-practice-minikube3](/images/kube-practice-minikube3.png)

도커 컴포즈를 활용한 배포는 워드프레스 컨테이너 하나와 MySQL 컨테이너 하나를 띄우고 각 포트와 환경변수를 설정해주면 된다.

```python
version: "3"

services:
  wordpress:
    image: wordpress:5.9.1-php8.1-apache
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: password
    ports:
      - "30000:80"

  mysql:
    image: mariadb:10.7
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: password
```

호스트의 퍼블릭 IP에 포트 30000번으로 엑세스하면 워드프레스 컨테이너의 포트인 80번 포트로 연결된다. 워드프레스 컨테이너는 내부 환경설정을 통해 DB를 MYSQL 컨테이너와 연결되도록 설정되어 있다.

MySQL 컨테이너의 경우 아무런 설정없이 MySQL 이미지만 줘도 기본 포트인 3306포트가 노출되어 있다. 그리고 도커 컴포즈의 컨테이너끼리는 별도의 커스텀 네트워크를 생성하여 지정해주지 않았다면 디폴트 네트워크로 연결되어 있다.

또한 같은 네트워크에 있는 컨테이너간의 통신은 서비스의 이름이 호스트 명으로 사용된다. 그래서 워드프레스 컨테이너에서 환경설정을 통해 자신의 DB로 mysql을 지정해주기만하면 자연스럽게 MySQL 컨테이너에서 열려있는 3306포트로 연결할 수 있다.

## 쿠버네티스로 워드프레스 배포

![kube-practice-minikube4](/images/kube-practice-minikube4.png)

쿠버네티스로 배포할 때는 도커 컴포즈때보다 다양한 구성요소들이 필요하다. Service, Pod, Replicaset, Deployment 등이 모두 설정파일에 명시되어야 한다. 이들에 대한 세부적인 설명은 차후로 미루지만 pod이 쿠버네티스의 가장 작은 실행단위로 한개 혹은 그 이상의 컨테이너로 구성된다는 정보는 도커 컴포즈와의 비교를 위해 필요하다.

위 그림에서 보면 두개의 pod을 만들어 각각 워드프레스와 MySQL 컨테이너를 띄워놨다. 컴포즈는 컨테이너 2개를 만들었지만 쿠버네티스로 배포를 할땐 Pod을 두개만들어 그 안에 컨테이너를 띄우고 replicaset과 deployment로 이들을 감싸고 있다.

외부 클라이언트가 각 컨테이너에 접근하기 위해서는 nodeport라는 서비스를 통해 접근해서 쿠버네티스를 구성하는 노드들에 접근하고 내부적으로는 clusterIP를 통해 특정 pod으로 접속해야한다. 도커 컴포즈는 외부 포트와 내부 컨테이너 포트를 지정(ex. 30000:80)해주는 것으로 외부통신을 가능케 했지만, 쿠버네티스는 서비스라는 종류의 구성요소 중 nodeport와 clusterip를 설정하여 외부와의 통신이 가능해진다.

```python
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mariadb:10.7
          name: mysql
          env:
            - name: MYSQL_DATABASE
              value: wordpress
            - name: MYSQL_ROOT_PASSWORD
              value: password
          ports:
            - containerPort: 3306
              name: mysql

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: wordpress:5.9.1-php8.1-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
              name: wordpress

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
```

도커 컴포즈와 가장 크게 차이나는 것을 꼽자면 두 pod안에 있는 컨테이너 간의 통신이다. 도커 컴포즈는 별도로 포트를 지정하지 않아도 컨테이너 외부로 포트가 열려있고 서비스명을 사용하면 디폴트 네트워크를 통해서 컨테이너간 연결이 가능하다. 그런데 쿠버네티스는 pod간의 통신을 위해서는 clusterip를 만들어줘야한다. clusterip는 내부에 노출하는 ip로써 여러개의 pod 간 서비스 이름으로 통신하도록 한다.

가령 mysql 컨테이너를 띄운다고 할때 도커 컴포즈는 아무런 설정이 없이 이미지만 줘도 mysql의 3306이라는 디폴트 포트가 노출된다. 하지만 쿠버네티스는 clusterip로 포트를 지정해주어야한다. 즉, mysql 이미지와 더불어 service에서 mysql을 바라보는 3306 포트를 추가로 지정해주어야한다.

```python
.
.
.

apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

.
.
.
```

위의 설정은 전체 설정 중 clusterip와 관련한 설정만 가져온 것이다. 서비스가 app이 wordpress. tier가 mysql인 pod를 바라보도록 selector를 설정하고 포트는 3306을 바라보게 설정을 한다. 이를위해 pod에 띄울 컨테이너 설정할 때 container port라는 옵션을 기입한다.

![kube-practice-minikube5](/images/kube-practice-minikube5.png)

좀더 쉽게 접근하자면, WAS들 앞에 Nginx와 같은 webserver를 두는 것과 개념이 비슷하다. 즉, 여러개의 pod 앞에 서비스(clusterip)를 두고 이 서비스를 통해 각 pod에 접근할 수 있다. 외부 클라이언트가 nginx를 통해 was에 엑세스하듯 서비스를 통해 pod에 엑세스하고 pod 간에 통신을 할 때도 이 서비스를 거쳐 통신하게 된다.
