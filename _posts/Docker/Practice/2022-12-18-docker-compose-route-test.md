---
title: Docker Practice - 도커 컴포즈 커맨드 파일 우선순위 테스트
categories: [Docker, Practice]
---

# 도커 컴포즈 파일이 여러개 있다면 무엇을 먼저 실행할까? 아니면 모두 실행될까?

docker compose 파일은 `docker compose up` 이라는 커맨드로 실행가능하다. 그런데 여러개의 docker compsoe 파일이 여러 경로에 나누어져 있다면 `docker compose up` 커맨드는 무엇을 먼저 실행시킬까?

이 의문을 갖게된건 특정 디렉토리에서 docker compose 파일을 작성하고 `docker compose up` 커맨드로 실행시킬려고 했는데 이전에 작성한 docker compose 파일이 자꾸 실행이 되는 현상을 발견하면서 부터이다.

그래서 디폴트 경로가 따로 있는건지 아니면 다른 우선순위가 정해져 있는 것인지 확인을 하고자 테스트를 진행했다.

## 테스트 환경 구축

우선 docker compose 파일들 중 가장 최상위에 위치한 docker-compose 파일은 홈디렉토리 바로 아래에 위치해 있다. 그리고 docker-compose-dir이라는 디렉토리 내에서 test 1~4까지의 디렉토리를 추가로 만들고 그안에 각기 다른 내용이 작성된 docker compose 파일을 위치시켰다.

![docker-practice-compose-command-test0](/images/docker-practice-compose-command-test0.png)

## 테스트 진행

- /home/won21yuk/docker-compose.yml

디렉토리 트리 상에서 가장 최상위에 위치한 docker compose 파일이 위치한 곳에서 실행하면 해당 파일이 실행된다.

- /home/won21yuk/docker-compose-dir

그럼 docker compose dir 디렉토리에서는 어떨까?

실행해보면 최상위에 위치한 docker compose 파일이 똑같이 실행된다. 그런데 이를 확장해서 test1~4 디렉토리로 이동해서 `docker compose up` 커맨드를 실행하면 조금 다르다.

test2와 test4에서는 최상위에 위치한 docker compose 파일이 실행되지만, test1과 test3 디렉토리에서는 해당 파일에 위치한 docker compose 파일이 실행된다.

차이를 보면 test2,4 에서는 docker compose 파일 이름이 docker-compose.yml이지만 test1,3은 그렇지 않다.

그럼 내가 현재 위치해 있는 곳은 docker compose 커맨드를 실행시킬 때 고려사항임이 확실하며 파일의 이름 또한 중요한 고려사항임을 알 수 있다.

그런데 각기 다른 디렉토리에 저장된 docker compose 파일은 바로 실행할 수 없는 걸까?

```python
Usage:  docker compose [OPTIONS] COMMAND

Docker Compose

Options:
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --env-file string            Specify an alternate environment file.
  -f, --file stringArray           Compose configuration files
      --profile stringArray        Specify a profile to enable
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name

Commands:
  build       Build or rebuild services
  convert     Converts the compose file to platform's canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service.
  down        Stop and remove containers, networks
  events      Receive real time events from containers.
  exec        Execute a command in a running container.
  images      List images used by the created containers
  kill        Force stop service containers.
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding.
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service.
  start       Start services
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
```

위에 작성된 내용은 docker compose -help로 옵션들을 확인한 내용을 가져온 것이다. -f 옵션을 사용하면 docker compose 파일을 지정해서 compose 할 수 있다.

이를 실행해보면 docker-compose-dir에서 test2에 들어있는 docker compose 파일을 -f 옵션을 사용해서 실행하면 현재 나의 위치에서 해당 파일을 찾을 수 없다는 에러 메세지가 출력된다.

![docker-practice-compose-command-test1](/images/docker-practice-compose-command-test1.png)

즉, 결국 -f 옵션을 사용하려면 내가 실행시키려는 docker compose 파일이 위치한 곳에 직접 이동해서 사용해야 했다.

![docker-practice-compose-command-test2](/images/docker-practice-compose-command-test2.png)

그럼 경로를 지정해주면 안될까하는 의문이 자연스럽게 생겼다. 경험상 옵션에 파일이름만 입력해서 커맨드를 입력하는 경우는 그 디렉토리에 위치했을 때를 기준으로 작동했었다. 그래서 상대 경로로 작동시켜도 문제가 없을 것이라고 생각이 들었다.

![docker-practice-compose-command-test3](/images/docker-practice-compose-command-test3.png)

결론적으로 **상대 경로를 지정하는 방식은 제대로 작동**한다. 내 위치를 기준으로 파일을 지정해주기면 하면 `docker compose -f` 커맨드가 실행된다. 그리고 당연하게도 절대경로로 사용해도 문제가 없다.

# 파일이름은 반드시 docker-compose.yml 이여야하나?

한가지 더 궁금한 점은 yml파일의 이름을 반드시 docker-compose 형태로 작성해야하는가 하는 부분이였다. 앞선 test1,3에 위치한 docker compose 파일을 인식한다는 게 사실은 docker-compsoe.yml 이라는 파일을 디폴트로 인식하게끔 되어 있는 것이 아닐까 하는 생각이 들었기 때문이다.

## 테스트 환경 구축

우선 최상위에 위치했던 docker-compose.yml 파일을 test.yml로 바꿔줬다. 그리고 test3에 test3-1 디렉토리를 생성하고 새롭게 docker-compose.yml 파일을 작성했다. 이는 test3-1 파일에 위치한 docker compose 파일을 test3에 바로 하위에 위치시켜놓기 위함이다.

![docker-practice-compose-command-test4](/images/docker-practice-compose-command-test4.png)

이로써 병렬로 위치한 docker-compose.yml 파일과 하위 디렉토리에 위치한 docker-compose.yml 파일이 있도록 구성되었다.

## 테스트 진행

test.yml이 있는 홈디렉토리에서 실행을하면 파일을 찾을 수 없다는 메세지가 뜬다. 이는 docker-compose-dir 디렉토리에서도 마찬가지다

![docker-practice-compose-command-test5](/images/docker-practice-compose-command-test5.png)

이로 유추할 수 있는 건 현재 내가 docker compose up **커맨드를 실행한 위치에서 상위로 읽어나가면서 docker-compose.yml 파일을 찾는다는 것**이다.

이를 좀더 명확하게 하기 위해서 test3 디렉토리로 이동해서 커맨드를 실행해보면, 해당 디렉토리에 위치한 파일이 실행된다.

![docker-practice-compose-command-test6](/images/docker-practice-compose-command-test6.png)

그리고 test3-1로 이동해서 실행해보면 test3-1에 위치한 파일이 실행된다.

![docker-practice-compose-command-test7](/images/docker-practice-compose-command-test7.png)

이제 좀 결론이 뚜렷해진다.

- docker-compose.yml이라는 이름을 가진 파일을 디폴트로 인식한다
- 현재 위치한 경로에 docker-compose.yml 파일이 없으면, 상위 디렉토리를 탐색해서 실행한다.

실제로 test3-1에 있는 파일을 testtest.yml로 이름을 바꾸고 docker compose up 커맨드를 실행하면 test3에 있는 docker-compose.yml 파일이 실행된다.

![docker-practice-compose-command-test8](/images/docker-practice-compose-command-test8.png)

# 결론

결국 내가 경험한 docker compose up 커맨드를 실행했더니 이전에 작성해둔 docker compose 파일이 실행되는 현상은 내가 docker compose 파일이름을 docker-compose.yml 이 아닌 docker-compose-guestbook.yml로 설정했었기 때문에 발생했던 것이였다.

즉 당시에 해당 디렉토리에서 docker-compose.yml 파일을 찾지 못하니 상위 디렉토리인 홈디렉토리에서 테스트용으로 작성해둔 파일을 찾아서 실행한 것이였다.

이 모든 것들을 간략하게 정리하면 아래와 같다.

- docker compose up 커맨드는 디폴트로 docker-compose.yml 파일을 찾아서 실행한다.
    - 단, 현재 경로에 없으면 상위 디렉토리에서 docker-compose.yml 파일을 찾아서 실행한다.
- docker compose 파일은 yml 양식으로 작성하면 모두 실행가능하다.
    - 단, -f 옵션을 사용하여 파일의 경로를 직접 지정해 docker compose 커맨드를 실행해야한다.
    - ex) docker compose -f test3/docker-compose-guestbook.yml
