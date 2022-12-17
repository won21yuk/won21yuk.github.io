---
title: Trouble Shooting - found character that cannot start any token
categories: [Docker, Trouble Shooting]
---
# 에러 : "yaml: line 3: found character that cannot start any token"

![docker-tr2-0](/images/docker-tr2-0.png)

docker compose 파일을 실행시킬려고하니 에러가 하나 뜬다. 해석이 뭔가 완전히 되지는 않는데 파일을 실행시킬 수 없게 만드는 문자가 yml파일안에 있다는 뉘앙스로 해석이된다. 그래서 처음에는 뭔가 오타가 있었나 싶어서 찾아봤는데 오타나 문법상의 문제는 없는 듯했다.

그래서 결국 구글링의 힘을 좀 빌렸는데 아주 심플한 이유면서 놓치기 쉬운 것이였다. 바로 yaml 파일은 tab키로 들여쓰기를 하면 인식이 안된다는 것이였는데 실제로 tab으로 들여쓰는 대신 스페이스바로 들여쓰기를 적용하기만 하면 바로 해결이 가능했다.

- Before

![docker-tr2-1](/images/docker-tr2-1.png)

- After

![docker-tr2-2](/images/docker-tr2-2.png)

결과적으로 실행시켰더니 잘 됐다. 기본적으로 yaml파일 자체가 미숙해서 발생한 에러지만 놓쳤던 기초 개념을 이제라도 바로잡아서 다행이라 생각한다.

![docker-tr2-3](/images/docker-tr2-3.png)
