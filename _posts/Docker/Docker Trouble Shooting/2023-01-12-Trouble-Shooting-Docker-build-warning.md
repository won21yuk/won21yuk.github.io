---
title: Trouble Shooting - python 기반의 docker 이미지 build 과정에서 발생한 Warning message
categories: [Docker, Docker Trouble Shooting]
---

# 개요

![docker-tr3-0](/images/docker-tr3-0.png)

python 이미지 기반으로 dockerfile을 작성하여 이미지로 bulid하는 과정에서 두개의 Warning Message를 직면했다. 해당 warning message가 결과적으로는 이미지를 생성하는데 영향을 미치진 않았지만, warning message를 보고도 넘길만한 쿨함이 없기 때문에 이에 관한 trouble shooting을 작성하게 되었다.

아래의 dockerfile은 초기에 작성했었던 내용을 담고 있으며, 해당 dockerfile을 build시키는 과정에서 warning message를 만났다.

```python
# dockerfile(origin)
FROM python:3.7-slim

WORKDIR /app

ADD . /app

RUN pip install -r requirements.txt

ENV GOOGLE_APPLICATION_CREDENTIALS="/app/docker-kubernetes-370811-f49003f1ee92.json"

CMD ["python", "tweepy-test"]
```

# Warning Message 1

---
💡 Warning : Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: [https://pip.pypa.io/warnings/venv](https://pip.pypa.io/warnings/venv)

---


해당 경고 메세지는 ‘*루트 유저로 pip를 실행시키는 것이 권한의 손상을 초래하거나 시스템 패키지 매니저와의 동작에서 충돌을 초래할 수 있다*’고 한다.

사실 해당 메세지가 완전히 처음보는 유형의 경고메세지는 아니다. 일반적으로 호스트상에 파이썬을 설치하는 경우 루트 권한(가령 sudo)으로 파이썬의 패키지 매니저를 실행시키지 않는 것을 권장하는 내용의 글을 stackoverflow, reddit 등의 개발자 커뮤니티에서 쉽게 접할 수 있었기 때문이다. 다만 이에 대한 내용은 꽤나 긴 내용이 될것이기 때문에 차후의 포스팅으로 미뤄두고 본 포스팅에서는 해당 경고 메세지를 해결하기위해 dockerfile을 어떻게 작성해야하는 지에 대해서만 다루도록 한다.

다시 경고 메세지로 돌아가면, 해결책으로 가상환경을 사용하는 것을 직접 추천하고 있다. 따라서 dockerfile에서도 가상환경 사용하는 방식으로 작성하면 해결할 수 있다고 이해했다.

가상환경을 사용한 dockerfile의 작성과 관련한 샘플 코드는 [여기서](https://testdriven.io/blog/docker-best-practices/#using-python-virtual-environments) 찾을 수 있었다.

builder 역할로 가상환경이 설치된 이미지를 생성하고 이렇게 생성된 builder 이미지의 가상환경을 copy해서 원하는 이미지를 생성하는 내용을 담고 있다. 이러한 방식을 multi-stage build라고 한다.

multi-stage build는 컨테이너 이미지를 만들면서 빌드에는 필요하지만 최종 컨테이너 이미지에는 필요 없는 환경을 제거할 수 있도록 단계를 나누어서 기반 이미지를 생성하는 것을 의미한다. 이는 docker를 사용하는데 있어 불필요하게 컨테이너 이미지가 커지는 것을 막기 위해 고안한 테크닉이다.

이를 반영하여 수정한 dockerfile은 아래와 같다.

```python
# dockerfile(edit1)

# temp stage
FROM python:3.7-slim as builder

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install -r requirements.txt

# final stage
FROM python:3.7-slim

COPY --from=builder /opt/venv /opt/venv

WORKDIR /app
ADD . /app

ENV PATH="/opt/venv/bin:$PATH"
CMD ["python", "tweepy-test.py"]
```

해당 dockefile의 실행 결과를 확인해보면  test라는 이름를 가진 최종 컨테이너 이미지와 <none>이라고 된 무명(無名)의 builder용 이미지가 생성된 것을 확인할 수 있다. builder용 컨테이너 이미지는 test라는 최종 컨테이너 이미지가 생성된 시점부터는 필요가 없다.

![docker-tr3-1](/images/docker-tr3-1.png)

# Warning Message 2

---
💡 WARNING: You are using pip version 22.0.4; however, version 22.3.1 is available. You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.

---

두번째 경고 메세지는 ‘현재 pip 버전을 22.0.4버전이지만 상위 버전인 22.3.1이 사용가능하다’는 내용이다. 이를위해 pip 업그래이드를 고려하라는 메세지도 출력된다.

따라서 dockerfile에 pip 업그래이드를 위한 `RUN pip install --upgrade pip` 를 추가해준다.

```python
# dockerfile(edit2)

# temp stage
FROM python:3.7-slim as builder

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN apt-get update && \
    apt-get install -y --no-install-recommends apt-utils gcc

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

# final stage
FROM python:3.7-slim

COPY --from=builder /opt/venv /opt/venv

WORKDIR /app
ADD . /app

ENV PATH="/opt/venv/bin:$PATH"
CMD ["python", "tweepy-test.py"]
```

여기서 한가지 신경써야할 부분은 pip를 실행시키는 커맨드 이전에 업그래이드를 진행시켜줘야한다는 점이다. pip 업그래이드 이전에 pip가 실행되면 똑같이 pip를 업데이트하라는 메세지가 출력되기 때문이다.  이를 위의 예제를 통해 부연설명하자면, `RUN pip install -r requirements.txt` 에 앞서 `RUN pip install --upgrade pip`가 위치해야만 경고 메세지가 더 이상 출력되지 않는다.

# Extra : Debconf

![docker-tr3-2](/images/docker-tr3-2.png)

위의 두 경고메세지를 수정하고 새롭게 나타난 붉은 색 메세지이다. 자세한 내용은 알 수 없지만, apt-utils가 설치되지 않아 패키지 구성이 딜레이 되고 있다는 내용이니 apt-utils 설치하는 내용을 추가했다.

그러나 여전히 해결되지 않았고 관련된 내용을 찾아봤으나 원인에 대한 뚜렷한 내용을 찾지는 못했다. 다만 다행히도 같은 이슈를 겪은 사람들이 토론한 [github 멘션](https://github.com/phusion/baseimage-docker/issues/319)을 찾을 수 있었고 여기서 간단한 해결법을 찾을 수 있었다.

단지 dockerfile의 시작에 `ENV DEBIAN_FRONTEND noninteractive` 와 `ENV DEBCONF_NOWARNINGS="yes"` 만 추가해주면 된다.

```python
# dockerfile(edit3)

# temp stage
FROM python:3.7-slim as builder

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NOWARNINGS="yes"

RUN apt-get update && \
    apt-get install -y --no-install-recommends apt-utils gcc

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

# final stage
FROM python:3.7-slim

COPY --from=builder /opt/venv /opt/venv

WORKDIR /app
ADD . /app

ENV PATH="/opt/venv/bin:$PATH"
CMD ["python", "tweepy-test.py"]
```

# Reference

[Multi-stage builds - Docker Documentation](https://docs.docker.com/build/building/multi-stage/)

[python - WARNING: Running pip as the 'root' user - Stack Overflow](https://stackoverflow.com/questions/68673221/warning-running-pip-as-the-root-user)

[Docker Best Practices for Python Developers - TestDriven.io](https://testdriven.io/blog/docker-best-practices/)

[[16.04] debconf: delaying package configuration, since apt-utils is not installed · Issue #319 · phusion/baseimage-docker (github.com)](https://github.com/phusion/baseimage-docker/issues/319)
