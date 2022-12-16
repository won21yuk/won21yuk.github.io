---
title: Docker Practice - 도커 설치 및 테스트
categories: [Docker, Practice]
---

실습은 Centos7이 설치된 GCP 인스턴스로 진행하며 root 계정을 사용합니다.

# 라이브러리 설치
도커와 도커 컴포즈를 설치한다. 도커 컴포즈는 yum으로 설치하면 구버전이 설치되기 때문에 직접 도커 깃허브에서 최신 버전을 다운 받는다.
```python
# docker 설치
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# docker compose 설치
curl -L https://github.com/docker/compose/releases/download/$(curl --silent https://api.github.com/repos/docker/compose/releases/latest | jq .name -r)/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose && chmod 755 /usr/bin/docker-compose

# 버전 체크
docker -v
docker-compose -v

```

# 도커 테스트1
mysql과 wordpress의 이미지를 가지고 와서 컨테이너를 띄운다.

- 일반 사용

```python
# mysql 설치
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  mysql:5.7

# in mysql : database 생성 및 권한부여
docker exec -it mysql mysql
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
quit

# wordpress 설치
docker run -d -p 8000:80 \
  -e WORDPRESS_DB_HOST=172.17.0.1 \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```

- 가상의 네트워크 사용
도커 네트워크를 생성하면 쉽게 컨테이너간 통신을 하도록 설정할 수 있다. `docker run` 커맨드를 사용할때 `--network={생성한 네트워크 이름}` 를 추가해주면 된다.

```python
# 네트워크 생성
docker network create app-network

# 네트워크에 연결된 컨테이너 생성(mysql + wordpress)
docker run -d \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  --network=app-network \
  mysql:5.7

docker exec -it mysql mysql
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
quit

docker run -d -p 8000:80 \
  --network=app-network \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```

# 도커 컨테이너 중지 및 삭제

```python
# 컨테이너 중지
docker stop {container name or id}

# 컨테이너 삭제 & 강제 삭제
docker rm {container name or id}
docker rm -f {container name or id}

# 도커 이미지 삭제
docker rmi {image id}

# 모든 컨테이너와 이미지 삭제
docker rm -f $(docker ps -aq)
docker rmi $(docker images -q)
```

# 도커 테스트 2 : 방명록 만들기
우선 방명록 페이지는 mongodb, gusetbook(frontend, backend) 이미지를 가져와서 3개의 컨테이너를 띄우고 앞서 테스트1에서 만든 app-network로 이들을 연결한다.

frontend는 8080포트를 열어서 외부에서 엑세스 가능하도록하며 내부적으로는 8000번포트로 backend와 통신한다. backend는 8000번 포트로 받은 데이터를 mongodb에 저장하는데 이때 mongodb의 기본 포트인 27017을 사용한다.

```python
# mongodb
docker run -d --name mongodb --network=app-network mongo:4

# guestbook backend
docker run -d --name backend \
  --network=app-network \
  -e PORT=8000 \
  -e GUESTBOOK_DB_ADDR=mongodb:27017 \
  subicura/guestbook-backend:latest

# guestbook frontend
docker run -d -p 8080:8000 \
  --network=app-network \
  -e PORT=8000 \
  -e GUESTBOOK_API_ADDR=backend:8000 \
  subicura/guestbook-frontend:latest
```

`인스턴스의 퍼블릭 ip:8080`으로 접속해서 방명록을 작성한 후 mongodb에 들어가서 확인하면 guestbook db가 생성되있는 것과 그 안에 messages라는 컬렉션이 생성된걸 볼 수 있다. 그리고 messages 컬렉션을 조회하면 앞서 작성한 방명록이 저장되어 있다는 것을 알 수 있다.

```python
# mongodb 컨테이너 접속
docker exec -it mongodb mongo

show dbs; # admin, config, local, guestboock
use guestbook;
show collections; # messages
db.messages.find();
```

![docker-practice-test](/imgaes/docker-practice-test.png)
