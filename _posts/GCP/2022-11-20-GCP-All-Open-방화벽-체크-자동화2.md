---
title: GCP Composer - GCP All-Open 방화벽 체크 자동화 (2)
categories: [GCP]
---

[지난 포스팅](https://won21yuk.github.io/posts/GCP-All-Open-%EB%B0%A9%ED%99%94%EB%B2%BD-%EC%B2%B4%ED%81%AC-%EC%9E%90%EB%8F%99%ED%99%94/)에서는 On-premise 환경에서 Airflow로 GCP All Open 방화벽 체크 자동화를 진행했었습니다. 이번 포스팅에서는 GCP에서 제공하는 Airflow 솔루션인 Composer을 사용하여 같은 작업을 진행해보고 Composer와 Github repo를 연결까지 해볼 계획입니다.

# GCP composer 시작하기

Cloud Composer를 사용하기 위해서는 composer API 사용 등록을 해줘야합니다. gcp 검색창에 composer 치면 바로 찾을 수 있습니다. 사용등록이 완료되면 아래와 같은 화면이 뜹니다.

![composer0](/images/composer-firewall-check0.png)

Composer 1은 클러스터 내의 노드의 수평확장을 수동으로 처리하고 Composer 2는 자동으로 처리합니다. 따라서 Composer 1는 Static한 상황에서 사용하는 것이 좋고 이때문에 저같이 소소하게 가지고 노는 용도라면 당연히 Composer 1이 낫습니다. 물론 사용 환경이 더 다이나믹하면 Composer 2가 편리 하겠지만, 그만큼 비용이 비쌉니다. 저는 당연하게도 Composer 1을 사용합니다. 자세한 비교는 [공식 홈페이지](https://cloud.google.com/composer/docs/composer-2/composer-versioning-overview)에서 확인할 수 있습니다.

![composer1](/images/composer-firewall-check1.png)

그 후 composer 환경을 구축하기 위한 설정을 이것저것 해줘야합니다. 노드 수, region, 머신유형, 디스크 크기, Oauth 범위 등 자신의 상황에 맞게 설정해주면 됩니다. 만들기를 누르면 composer 환경이 생성되기 시작하는데 이 작업이 꽤 오래걸립니다. 전 10분 넘게 걸린 듯 하네요.

![composer2](/images/composer-firewall-check2.png)

그다음엔 파이썬 라이브러리들을 composer 환경에 설정해야줘야 합니다. 사용하고자하는 라이브러리 이름을 패키지 이름에 넣고, 버전을 지정하고 싶다면 `[extra]==3.0.1`과 같은 방식으로 옆 칸에 적으면 됩니다. 그리고 저장을 누르면 적용이되는데 이것도 5분이상 걸립니다.

![composer3](/images/composer-firewall-check3.png)

다음은 airflow 옵션을 설정해줘야합니다. airflow.cfg파일을 이전에는 직접 열어서 수정했지만 Composer를 사용하면 콘솔 상에서 모든것이 가능합니다. 최종적으로 제가 한 설정은 아래의 사진과 같습니다. 다만 이것도 5분 이상 걸립니다….

![composer4](/images/composer-firewall-check4.png)

기본 설정이 끝났다면 가장 먼저해야하는 건 당연히 전에 작성한 Dag파일과 함수 파일을 업로드 하는 것입니다. Composer 환경을 구축하면 자동으로 Cloud Storage에 버켓을 만들어서 그곳에 바로 업로드 할 수 있는 구조라 매우 편리합니다.

![composer5](/images/composer-firewall-check5.png)

다만 업로드 후에 Dag에서 함수 파일(dag_funcs.py)을 import하기 위한 경로 지정에서 문제가 있습니다.

![composer6](/images/composer-firewall-check6.png)

우선 GCS 버켓에 경로가 어떻게 잡히는 지를 먼저 확인해봤습니다.

```python
# test_dag.py
import os

from airflow import DAG
from airflow.operators.python import PythonOperator
from pendulum import yesterday

dag = DAG(
    dag_id="test",
    schedule_interval=None,
    start_date=yesterday("Asia/Seoul")
)

def test_callable():
    print(os.getcwd())
    print(os.path.abspath(__file__))

test = PythonOperator(task_id='test',
                      python_callable=test_callable,
                      dag=dag)

# dependency
test
```

- os.getcwd : `/home/airflow`
- os.path.abspath : `/home/airflow/gcs/dags/test_dag.py`

경로를 알아내고 다양한 방식으로 import 경로를 지정해봤습니다. 그런데 sys, os를 사용해도 deafult 경로가 dags 폴더에서 변하질 않는다는 것을 확인했습니다. 그럼 이 default 경로를 변경하던지 아니면 맘편하게 utils 폴더를 dags 폴더안에 넣어버리던지 해야했는데, default 경로를 바꾸는 방법을 찾지 못했고, 어쩔수없이 utils 폴더를 dags 폴더 밑으로 집어넣었습니다.

![composer7](/images/composer-firewall-check7.png)

그 결과 다행히도 정상적으로 인식이 되긴 합니다. 뭔가 원하는대로 하질 못해서 찝찝한 감이 없잖아 있지만 우선 정상적으로 인식됐다는 사실에 만족합니다. 이제 dag가 정상작동하는지만 확인해주면 됩니다. 물론 airflow connection에서 slack webhook 설정을 마친 상태여야 합니다.

# Dag 트리거

![composer8](/images/composer-firewall-check8.png)

Dag를 트리거 시키니 정상 작동합니다. 그런데 로그를 보니 에러가 하나 발생했습니다.

![composer9](/images/composer-firewall-check9.png)

file_cache가 가능하기 위해서는 oauth2client의 버전이 4.0.0이상이면 안된다는 에러메세지인데, 이 에러가 task 실행에는 영향을 미치지 않지만 에러는 뜨는 묘한 상황입니다. 그래서 file_cache가 뭔지 궁금해서 좀 찾아봤는데 나오는 내용의 대부분이 저와 같은 에러가 발생해서 해결 어떻게 하는지에 대해 묻는 것들이고, 그나마 설명이 나오는게 AWS 블로그에서 올라온 글 하나입니다.

![composer10](/images/composer-firewall-check10.png)

얼추 읽어보니 AWS나 GCP 같은 클라우드 환경에서 API간 파일을 빠르게 읽기 위해서 네임스페이스 상에 파일을 캐싱하는 내용인듯합니다. Cloud Composer같은 경우 Cloud storage에서 dag 파일을 읽어서 실행되는 구조니 최초 dag 실행시 네임스페이스에 캐싱해놓고 스케쥴에 따라 dag를 실행할 때 바로바로 캐싱해서 실행하는 것이 매번 Storage에 가서 dag 파일을 읽는 것 보다 효율적일겁니다. 그래서 GCP에서 file cache를 사용하는 듯하고, 없어도 storage가서 파일들을 읽으면 되니 task에는 문제는 없는 그런 상황이라고 생각됩니다. 어쨌든 file cache를 사용하면 리소스 효율도 좋고 워크로드도 빨라지고 여러면에서 좋긴해보이니 저도 한번 사용해 보겠습니다.

![composer11](/images/composer-firewall-check11.png)

에러 메세지에 뜨는 대로 oauth2client를 4.0.0버전 아래로 다운그래이드 시켜줬습니다. 다운그래이드만 하면 되니 해결 자체는 너무 쉽습니다. 근데 성능이 빨라진건 못느끼겠습니다. 워낙 작은 워크플로우 하나만 덜렁있으니 효과가 있을리가 없죠. 그래도 뭐 좋은게 좋은거다 하면서 넘어가겠습니다.

# Github repo에 연결

![composer12](/images/composer-firewall-check12.png)

이미 Composer은 Cloud storage에 dag 폴더를 가지고 있기 때문에 repo를 직접 연결할 수는 없고 대신 Cloud build를 사용하여 sync되도록 하겠습니다.

제일 먼저 해야하는 건 github repo를 생성해주는 것입니다. 저는 이전 포스팅에서 작성한 파일들을 push 해놨던 repo를 그대로 사용합니다.

![composer13](/images/composer-firewall-check13.png)

그 다음은 yaml 파일을 작성해야합니다. 이 파일은 sync하는 방식을 설정하는 파일입니다.

```python
# cloudbuild.yaml
steps:
- name: ubuntu
  args: ['bash', '-c', "echo '$COMMIT_SHA' > REVISION.txt"]
- name: gcr.io/cloud-builders/gsutil
  args:
    - '-m'
    - 'rsync'
    - '-d'
    - '-r'
    - 'dags'
    - 'gs://asia-northeast1-firewall-ch-b8007ea5-bucket/dags'
- name: gcr.io/cloud-builders/gsutil
  args:
    - '-m'
    - 'rsync'
    - '-d'
    - '-r'
    - 'dags/utils'
    - 'gs://asia-northeast1-firewall-ch-b8007ea5-bucket/dags/utils'
options:
  logging: CLOUD_LOGGING_ONLY
```

GCS의 bucket에서  dags 폴더와 utils 폴더만 sync 되도록 설정했습니다. 그리고 이 파일을 repo의 최상위 경로에 위치시키면 됩니다.

다음은 cloud build의 설정을 할 차례입니다. 일단 트리거를 만들어줘야합니다. 소스 - 저장소 - 새 저장소의 연결에 들어가서 자신의 github repo와 연결해주면 됩니다.

![composer14](/images/composer-firewall-check14.png)

그리고 유형을 Cloud build 구성파일로 변경하고 대체변수에는 _GCS_BUCKET을 넣어주면 됩니다. 그러면 설정은 끝납니다.

![composer15](/images/composer-firewall-check15.png)

그런데 push 테스트를 해봤더니 region관련 에러가 발생합니다. 그래서 다른 지역도 해봤더니 똑같이 에러가 발생합니다.

![composer16](/images/composer-firewall-check16.png)

결국 전역으로 REGION으로 바꿔주니 해결됩니다. 다른 블로그에서도 같은 증상이 발생했다고하는데 그 블로그들도 저도 이 현상의 원인을 찾지는 못했습니다.

# 마무리

맨날 on-premise에서 빌드해서 사용하다가 인생에서 처음 Saas를 사용해봤는데 왜 사람들이 사스~사스~하는지 알거같습니다. 사용해보니 다 갖춰져있으니 그냥 조금의 설정만 하고 사용하기만 하면되니까 진짜 편하긴 합니다. 다만 팔자 좋은 소리로 뭔 설정 하나만 수정해도 최소 5분씩 기다려야하니 좀 몸이 근질근질합니다.

어쨌든 데이터 엔지니어 입장에서 보면, 온전히 파이프 라인에 집중하기 위해선 인프라랑 시스템 업무가 어느정도 구분이 되어야한다고 생각이 드는데, 결국 이런 면에서 composer와 같은 Saas들을 어느정도 익혀두는건 데이터 엔지니어를 꿈꾸는 저에게 충분히 발전적인 방향이라고 생각이 듭니다. 앞으로 다른 여러 Saas들을 한번씩 사용해보는 시간들을 가져봐야 겠다는 다짐을 가지고 이만 글을 줄이겠습니다.

# Reference

[Cloud Composer 시작하기 – Changhyun Lee – Fake Nerd (chang12.github.io)](https://chang12.github.io/cloud-composer-start/)

[Cloud Composer 버전 관리 개요  |  Google Cloud](https://cloud.google.com/composer/docs/composer-2/composer-versioning-overview)

[Cloud Composer 1과 2의 차이점은 무엇인가요? : BESPIN GLOBAL Support Portal](https://support.bespinglobal.com/ko/support/solutions/articles/73000588870-cloud-composer-1%EA%B3%BC-2%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94-)

[GCP Composer를 Git repo에 연동 (tistory.com)](https://burning-dba.tistory.com/152)

[Google Cloud Platform의 Cloud Composer를 이용한 Workflow Orchestration - BESPINGLOBAL](https://www.bespinglobal.com/gcp-workflow-orchestration/)
