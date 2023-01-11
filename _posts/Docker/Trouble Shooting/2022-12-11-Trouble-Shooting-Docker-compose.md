---
title: Trouble Shooting - ERROR Version in "./docker-compose.yml" is unsupported.
categories: [Docker, Docker Trouble Shooting]
---

docker-compose 커맨드를 실행시키려는 중에 에러가 하나 발생했다.

![docker-tr1-0](/images/docker-tr1-0.png)

에러메세지를 읽어보면 docker-compose의 version 문제인듯하다. 해당 문제를 해결하기 위해 기존 docker-compose 버전을 삭제하고 최신 버전을 설치해주자. 최신 버전은 [도커의 깃허브](https://github.com/docker/compose/releases/)에서 받아 올 수 있다.

```python
# docker-compose 삭제
yum remove -y docker-compose

# jq 설치
yum install -y jq

# 최신 버전 체크 및 설치 경로 지정
VERSION=$(curl --silent https://api.github.com/repos/docker/compose/releases/latest | jq .name -r)
DESTINATION=/usr/bin/docker-compose

# 최신 버전의 docker-compose 설치
curl -L https://github.com/docker/compose/releases/download/${VERSION}/docker-compose-$(uname -s)-$(uname -m) -o $DESTINATION

# docker-compose 디렉토리에 권한 부여
chmod 755 $DESTINATION

# 버전 체크
docker-compose -v
```

위의 과정을 한줄로 정리하면 아래와 같다.

```python
# 한 줄짜리 코드
curl -L https://github.com/docker/compose/releases/download/$(curl --silent https://api.github.com/repos/docker/compose/releases/latest | jq .name -r)/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose && chmod 755 /usr/bin/docker-compose

# 버전 체크
docker-compose -v
```

나는 루트 계정을 이용했기 때문에 sudo를 사용하지 않았지만 루트 계정을 사용하지 않는 사람이라면 sudo를 입력해줘야한다.

![docker-tr1-1](/images/docker-tr1-1.png)

결과적으로 docker-compose 커맨드가 정상 작동하는 것을 확인할 수 있다.

# Reference

[docker-compose 최신버젼 설치 (velog.io)](https://velog.io/@nohsangwoo/docker-compose-%EC%B5%9C%EC%8B%A0%EB%B2%84%EC%A0%BC-%EC%84%A4%EC%B9%98)

[docker-compose 에러 Version in "./docker-compose.yml" is unsupported (tistory.com)](https://bug41.tistory.com/116)
