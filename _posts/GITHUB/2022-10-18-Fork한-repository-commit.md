---
title: fork한 repository에 commit해도 잔디가 안심어지는 문제
categories: [Github]
---

오늘 열심히 깃헙 블로그 글을 작성하고, 잔디를 봤는데 잔디가 채워지지 않았다.

뭔가 싶어서 이전 것도 보니, 내 repository에 commit은 반영이 되어있는데 잔디만 안 심어져있었다.

그래서 여기저기 검색해보니, fork한 repository에 commit하면 잔디심기가 안된다고 한다.

내가 잔디 심기에 집착하는건 아니지만, 막상 잔디가 안심어졌다하니 난 뭐한건가 싶어서 왠지 모를 공허함이 생겼다.

잃어버린 내 잔디가 심어지면, 이 공허한 마음 채워질지 한번 확인해보도록하겠다.

# 해결법
github에 잔디를 심기위해서는 크게 두가지 조건이 충족되어야하는데,
1. github 계정에 등록된 이메일이 commit할 때 등록된 user.email과 같아야하고,
2. 내 소유의 repository여야한다. 이는 **fork한 repository면 안된다는 뜻**과 같다.

# 1. user email을 github 계정과 일치 시키기
내 경우는 2번에 해당하지만 그래도 정리해놓는 김에 1번 해결법도 적어 놓으려 한다.

user.email과 name을 확인하는 방법은 아래의 명령어를 실행하면 된다.

```bash
git config --list
```
위 명령어를 사용하면 다음과 같이 config 정보들이 출력된다. 안보이면 엔터치면 한줄씩 더 볼 수 있다.

```bash
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=C:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=true
pull.rebase=false
credential.helper=manager-core
credential.https://dev.azure.com.usehttppath=true
init.defaultbranch=master
user.email=won21yuk@gmail.com
user.name=won21yuk
```

## config 설정 변경하기
user.name이나 user.email이 github 계정과 다른 경우는 직접 변경하는 커맨드를 입력해주어야한다.

깃허브 계정을 하나만 쓴다면, --global 옵션을 통해 루트 디렉토리의 설정을 한번에 변경해주는 것이 좋다.

다른 repository 마다 일일이 config 설정을 변경 안해도 때문이다.

```bash
git config --global user.name "song"
git config --global user.email "song@gmail.com"
```
하지만 config 설정만 변경하면 이전에 commit한 것들이 잔디심기에 반영이 되지않는다.

나의 목표는 잔디를 되살리는 것이기 때문에 이에 만족할 수 없다.

## 이전에 커밋한 것을 잔디밭에 다시 심어주기
이전에 commit한 내용을 잔디 심기하기 위해선 꽤나 번거로운 과정을 반복해야한다.

### 1. 이전 commit의 해쉬코드 찾기

```bash
git log --pretty=format:"%h = %an , %ar : %s" --graph
```

해당 커맨드를 입력하면 아래와 같이 commit들의 해쉬코드들이 출력된다.

```bash
* a5113ac = won21yuk , 64 minutes ago : 2022-10-17 HDI ch3 upload
* c73a881 = won21yuk , 13 hours ago : 2022-10-17 Implementation2 upload
* 6fbcc23 = won21yuk , 2 days ago : 2022-10-15 implementation upload
* e366fa0 = won21yuk , 4 days ago : 2022-10-13 greedy 4 edit
* 5a80c11 = won21yuk , 4 days ago : 2022-10-13 greedy 4 upload
* daef075 = won21yuk , 4 days ago : 2022-10-13 HDI ch2-2 edit2
* e274f14 = won21yuk , 4 days ago : 2022-10-13 HDI ch2-2 edit
* e072b28 = won21yuk , 4 days ago : 2022-10-13 HDI ch2-2 upload
* caa56bf = won21yuk , 5 days ago : 2022-10-12 HDI context edit3
```

이 로그들에서 잔디를 심지 못한 commit들 중 가장 오래전인 commit의 해시코드를 적어놓고 다음 단계에서 사용해야한다.

### 2. rebase 하기

```bash
git rebase -i 해쉬코드
```

위의 커맨드를 입력하면, 아래와 같이 출력된다.

보면 알겠지만 입력한 해쉬코드 이후의 commit 리스트가 출력되기 때문에, 위에서 가장 오래전인 commit의 해시코드를 적어놔야한다고 했던 것이다.

```bash
pick e213481 2022-10-11 context adit
pick f998f71 2022-10-12 HDI ch2 Posting update
pick 91941e0 2022-10-12 HDI context edit
pick c0a0d2d 2022-10-12 HDI context edit2
pick a870550 2022-10-12 HDI context edit
pick caa56bf 2022-10-12 HDI context edit3
pick e072b28 2022-10-13 HDI ch2-2 upload
pick e274f14 2022-10-13 HDI ch2-2 edit
pick daef075 2022-10-13 HDI ch2-2 edit2
pick 5a80c11 2022-10-13 greedy 4 upload
pick e366fa0 2022-10-13 greedy 4 edit
pick 6fbcc23 2022-10-15 implementation upload
pick c73a881 2022-10-17 Implementation2 upload
```

여기서 수정해야하는 commit들은 pick 대신 edit으로 변경해줘야한다.

pick은 그대로 두겠다는 의미고, edit은 변경하겠다는 뜻이다.

다 변경해 주었으면, ecs + wq!로 해당 파일을 저장해준다.

### 3. 지정한 commit들의 email 변경해주기
이제 edit으로 설정해둔 commit들의 email을 나의 github 계정에 등록된 email로 변경해줘야 한다.

```bash
git commit --amend --author="이름 <본인 이메일>"
```

위의 커맨드를 입력하면 이런저런 글자들이 터미널에 출력될텐데 아무것도 건들지말고 그냥 바로 q를 입력해 나와주면 된다.

그 다음 아래의 명령어를 입력해서 다음 commit으로 넘어간다.

```bash
git rebase --continue
```

이제부턴 반복이다.

다시 email 변경해주고 q로 나오고 continue해주고를 반복하면 된다.

이 노가다는 'fetal: No rebase in progress?'라는 문구가 뜨면 끝이 난 것이다.

마무리로 내 repository로 강제 push를 진행해주면 된다.

```bash
git push origin +master
```

master 말고 branch로 보내고 싶다면 해당 branch 명을 입력해주면 된다.

다만, **'+'는 반드시 써줘야합니다.** 이게 강제로 push 시켜주는 기호기 때문이다.

만약 +를 안써주면 다시 끔찍한 노가다를 다시해야할 수 도 있으니 주의해야한다.

# 2. fork한 repository를 없애고 새로 만들기
내가 직면한 문제가 바로 이것이다.

나는 github page로 블로그를 만들 때, 블로그 템플릿을 fork해서 가져와서 만들었기 때문이다.

해결방법은 위 1번보다 훨씬 간단하다.

### 1. 새 repository 만들기
내 소유의 repository를 만들기 위해서 일단 새 repository를 만들어 준다.

### 2. mirror clone하기
이전에 fork한 repository를 mirror clone 해줄 것이다.

clone 커맨드의 --mirror 옵션은 commit 이력도 clone하게 해준다.

commit을 살리는게 목표기 때문에 mirror clone을 사용해야 한다.

(사실 commit만 살리고자하는 거라면 bare 옵션을 사용해도 무관하다.)

```bash
git clone --mirror https://github.com/won21yuk/ex-repository.git
```

이러면 작업하고 있는 디렉토리에 해당 repository가 복사된다.

### 3. mirror push 하기
복사된 repository가 있는 디렉토리로 이동해주고, mirror push를 해준다.

만약 새 repository의 이름이 이전과 다르다면, remote로 새 repository의 url을 지정해 주고 mirror push 해야한다.

(mirror clone한 이후, 이전 repository를 삭제하고 새 repository 이름을 예전 repository 이름으로 변경하였다면 remote 등록해줄 필요는 없다.)

```bash
cd ex-repository.git
git remote set-url --push origin https://github.com/won21yuk/my-repository.git
git push --mirror
```

이러면 이전 commit들이 정상적으로 잔디로 심어진 것을 확인할 수 있다.




