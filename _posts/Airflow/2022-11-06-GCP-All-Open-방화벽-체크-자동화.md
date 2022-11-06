---
title: Airflow - GCP All-Open 방화벽 체크 자동화 (1)
categories: [Airflow]
---
*이전 [GCP의 방화벽 체크 및 슬랙 메세지 전송 포스팅](https://won21yuk.github.io/posts/GCP-All-open-firewall-check/)과 이어지는 포스팅입니다.*

이번 포스팅에서는 방화벽 체크를 자동화하기위해 Aiflow를 사용합니다.

이는 두가지 버전으로 나누어서 할 계획인데, 하나는 GCP 인스턴스에서 직접 Airflow를 설치하여 작동시키는 것이고 다른 하나는 Airflow로 워크플로를 만들고 배포하는데 도움을 주는 구글 클라우드의 composer 솔루션을 사용하는 것입니다.

이번엔 에어플로우를 직접 설치하여 DAG 작성하고 구동시키는 것까지 진행하고 다음 포스팅에서는 composer를 사용하도록 하겠습니다.

해당 포스팅에서 사용될 인스턴스의 정보는 아래와 같습니다.

> **인스턴스(airflow-test) 정보**   \
OS : Ubuntu 18.04   \
CPU : 2core   \
Ram : 4GB
>

# 파이썬 3.7 설치

기본적으로 GCP에서 ubuntu로 인스턴스를 생성하면, Python 3.6버전이 설치 되어있습니다.

그런데 저는 3.7버전으로 업그레이드하여 사용할 것입니다.

이유는 두 가지입니다.

첫째, GCP 방화벽을 체크하는데 사용하는 라이브러리가 파이썬 3.5~3.8버전에서 작동하는데 구글 클라우드에서는 특별히 3.7버전을 권장하고 있기 때문입니다.

둘째, Airflow 2.3 버전 이상은 3.7~3.10버전 사이의 파이썬과 호환되기때문에 3.7버전을 사용해야합니다.

```shell
# 최초 인스턴스 생성 후, 업데이트 및 업그레이드
sudo apt-get -y update &&\
sudo apt-get -y upgrade &&\
sudo apt-get -y dist-upgrade

# Deadsnakes PPA(Personal Package Archive) 레포지토리 만들기 및 등록
sudo apt install software-properties-common &&\
sudo add-apt-repository ppa:deadsnakes/ppa

# PPA등록 후 최신화를 위한 업데이트 & 업그래이드
sudo apt-get -y update

# 기본 명령어 일괄 설치
sudo apt-get install -y vim wget unzip ssh openssh-* net-tools

# python 3.7 설치
sudo apt-get -y install python3.7

# 설치된 위치 확인
which python3.7 #/usr/bin/python3.7

# python 패키지 관리목록에 등록 및 3.7버전 최우선 순위로 설정
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 2

# 패기지 관리 현황 확인
sudo update-alternatives --config python3

---------------------------------------------------------------------------

Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.7   2         auto mode
  1            /usr/bin/python3.6   1         manual mode
  2            /usr/bin/python3.7   2         manual mode

---------------------------------------------------------------------------

# pip 설치
sudo apt-get -y install python3-pip

# alias 설정
sudo vim ~/.bashrc

"""
	#python
	alias python=python3
	alias pip=pip3
"""

source ~/.bashrc

# python, pip 버전확인
python -V
pip -V
```

# Airflow 설치

```shell
# pip 업그레이드
python -m pip install --upgrade pip

# 필요 모듈,라이브러리 설치
python -m pip install cffi

sudo apt-get install -y --no-install-recommends \
        freetds-bin \
        krb5-user \
        ldap-utils \
        libffi6 \
        libsasl2-2 \
        libsasl2-modules \
        libssl1.1 \
        locales  \
        lsb-release \
        sasl2-bin \
        sqlite3 \
        unixodbc

# path 설정
sudo vim ~/.bashrc

"""
	#airflow
	export PATH=$PATH:/home/won21yuk/.local/bin
"""

source ~/.bashrc

# 에어플로우 설치(v 2.4.2)
python -m pip install apache-airflow

# airflow db 시작
airflow db init
```

db init을 하면, home 디렉토리에 airflow라는 디렉토리가 생깁니다.

![auto-firewall-check1](/images/auto-firewall-check1.png)

```shell
# dags,utils 폴더 생성
cd airflow
mkdir dags
mkdir utils

# user 생성
airflow users create \
          --role Admin \
          --username admin \
          --password 1234 \
          --firstname Song \
          --lastname won21yuk \
          --email won21yuk@gmail.com

# 에어플로우 설정
vim airflow.cfg

"""
# line 18
default_timezone = Asia/Seoul

# line 52
load_examples = False

# line 531
default_ui_timezone = Asia/Seoul

# line 963
dag_dir_list_interval = 60
"""

# airflow db 리셋
airflow db reset

# nohup명령어(터미널 세션이 끊겨도 동작하도록 하는 명령어) , & 명령어 (백그라운드 작동 명령어)로 켜기
nohup airflow webserver &
nohup airflow scheduler &

```

# 방화벽 설정

GCP 방화벽 규칙을 하나 만들어서 airflow webserver 기본포트인 8080을 열어줍니다. 기본포트를 사용하기 싫다면, airflow.cfg 설정에서 기본 포트 변경해주고 변경한 포트를 방화벽 규칙 생성때 입력하면 됩니다.

전 ip필터에 제 ip를 걸어둬서 제 ip로만 webserver 접속을 허용해놨습니다. 이는 필요에 맞게 설정하면 됩니다.

![auto-firewall-check2](/images/auto-firewall-check2.png)

그리고 인스턴스에 새롭게 만든 방화벽규칙(airflow) 태그를 추가해주면 방화벽 설정은 끝입니다.

![auto-firewall-check3](/images/auto-firewall-check3.png)

# DAG 작성

자동화에 대해서 생각해보면, 완전 자동화 failover를 구현할 수 없는 한 운영자의 개입이 필수불가결합니다. 그런 의미에서 에러상황 발생시, 운영자가 이를 빠르게 인지하고 대처할 수 있는 방안을 고려하는 것은 당연합니다.

보통 이를 위해 운영자에게 에러에 대한 메세지를 보내는 방식을 사용하는데, slack과 email이 흔히 사용됩니다.

근데 전 개인적으로 하나만 하기에는 뭔가 좀 아쉬운 감이 있어서 둘 중 하나만 선택하는 대신, 투 트랙 전략을 구상할 겁니다.

굳이 이유를 더 갖다붙이자면, 만에 하나 email만 확인가능한 상황에 있을 수도있고 마찬가지로 slack만 확인가능한 상황에 있을 수도 있으니 모든 경우를 고려하는 겁니다.

## 추가 설정

email과 slack으로 메세지를 전송해주기 위해서는 추가적인 설정이 필요합니다.

우선 email 전송을 위해 구글 계정의 2단계 인증을 활성화 시켜야합니다. 그 후 앱 비밀번호를 생성해줍니다.

![auto-firewall-check4](/images/auto-firewall-check4.png)

![auto-firewall-check5](/images/auto-firewall-check5.png)

그 다음은 airflow.cfg에서 여러 설정을 건드려줘야합니다.

```shell
# in airflow.cfg

# line 99
enable_xcom_pickling = True

# line 512
base_url = http://{인스턴스의 퍼블릭 ip}:8080

# line 749
# gmail과 연동하기 위한 설정
smtp_host = smtp.gmail.com
smtp_starttls = True
smtp_ssl = False
# Example: smtp_user = airflow
smtp_user = won21yuk@gmail.com
# Example: smtp_password = airflow
smtp_password = 앞선 과정에서 받은 앱 비밀번호 입력
smtp_port = 587
smtp_mail_from = won21yuk@gmail.com
smtp_timeout = 30
smtp_retry_limit = 5
```

이메일 관련 설정은 끝났으니 airflow webserver에서 slack connection을 생성해주어야합니다.

![auto-firewall-check6](/images/auto-firewall-check6.png)

여기서 중요한건 Host와 password 부분인데 이 두개는 incoming webhook URL을 사용해서 작성합니다. 전체 incoming webhook URL 중 https://hooks.slack.com/services/ 까지는 Host에 적어넣고, 뒤에 적힌 나머지 URL은 Password로 넣어주면 됩니다.

Connection ID는 이후 SlackWebhookOperator를 사용하여 http 통신할 때 사용합니다.

![auto-firewall-check7](/images/auto-firewall-check7.png)

마지막으로는 GCP 인스턴스를 수정해줘야하는데 이를 위해 우선 인스턴스를 정지시켜야합니다.

정지시킨 후 인스턴스 수정으로 들어가서 모든 Cloud API에 대한 전체 엑세스 허용으로 바꿔준 후 저장하면 됩니다.

이전 포스팅에서 Google Cloud API를 사용하기 위해 이것저것했던걸 내부시스템적으로 간편하게 처리하는 겁니다.

![auto-firewall-check8](/images/auto-firewall-check8.png)

## 최종코드

이제 코드를 작성해보겠습니다.

### airflow/dags/firewall_check_dag.py

dag 파일은 두개의 테스크를 사용했습니다. 그중 하나(t2)는 강제로 dag실패를 만들기위해 사용한 테스트 용 테스크이고 중요한 건 t1 테스크입니다.

t1테스크는 이전 포스팅에서 작성했던 방화벽 체크 코드를 실행시키는 테스크입니다. 이 코드가 정상 작동하는 지 여부에 따라 dag 전체의 성공 또는 실패와 직결되도록 했습니다.

이는 방화벽 체크 코드의 작동 여부를 사전에 체크하고 실패시 에러메세지를 바로 받아보기 위함입니다. 물론 정상 작동하면 해당 코드를 사용해 슬랙으로 all-open 방화벽 리스트 메세지를 보내게 됩니다.

에러 메세지를 보내는 방식은 앞서 언급했듯이 투트랙으로 구성했습니다. default_args의 email_on_failure 파라메터를 사용해서 DAG 실패시 email을 보내도록 하였고, 슬랙메세지는 callback파라메터를 사용했습니다.

여기서 좀 알아둬야할 건 email_on_failure와 callback 파라메터의 차이점입니다. email_on_failure는 전체 DAG를 기준으로 실패 시 이메일을 전송하고 callback 파라메터는 매 task마다 실패, 성공 여부를 판단하고 그에 맞춰 작성한 callback 함수가 실행됩니다.

이때문에 각자 상황에 맞춰서 잘 선택해야합니다. 저는 에러 테스트를 위해 t2 테스크를 사용했을 때 외에는 t1 테스크 하나만 작동하도록 했기 때문에 callback 파라메터를 사용했습니다.

또한 callback 파라메터는 on_failure_callback 와 on_success_callback 두가지로 나누어서 사용가능합니다. 저는 이를 활용해 task가 성공하면 정상적으로 방화벽 리스트를 슬랙에 전송하고, 실패하면 에러메세지를 전송하도록 했습니다.

(defulat_args의 파라메터에 대한 세세한 정보는 [공식 홈페이지](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/baseoperator/index.html)에서 확일 할 수 있습니다.)

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from pendulum import yesterday

import sys
import os
sys.path.append(os.path.dirname(os.path.abspath(os.path.dirname(__file__))))
from utils.dag_funcs import auto_firewall_check

# 인스턴스 선언
auto = auto_firewall_check()

# arguments 작성
default_args = {
    "owner": "won21yuk",
    "email": ["won21yuk@gmail.com"],
    "email_on_failure": True,
    "on_failure_callback": auto.on_failure_callback,
    "on_success_callback": auto.on_success_callback
}

# dag 선언
dag = DAG(
    dag_id="Automating_with_AIRFLOW",
		# 오늘부터 3일마다 15시에 작동하도록 스케줄링
    schedule_interval="0 15 */3 * *",
    start_date=yesterday("Asia/Seoul"),
    default_args=default_args
)

# 방화벽 체크가 정상 작동되는지 확인하는 테스크
# 방화벽 체크가 정상적으로 되는지에 따라 failure나 success callback함수 작동하도록하기 위함
t1 = PythonOperator(
        task_id="firewall_check",
        python_callable=auto.firewall_checks,
        dag=dag
        )
"""
# email과 slack으로 에러메세지 전송하기 위해 강제로 실패하도록 만든 테스크
t2 = PythonOperator(
        task_id="email",
        python_callable=auto.exception_method,
        dag=dag
        )
"""

# t1 >> t2
t1
```

### airflow/utils/dag_func.py

dag 작동을 위해 필요한 함수들은 모두 class로 묶어서 별도의 파이썬 파일로 작성했습니다. 이렇게 함수파일과 dag파일을 분리하는 방식은 dag파일의 가독성을 높혀주기 위해서 자주 사용됩니다.

```python
"""
pip install oauth2client
pip install google-api-python-client
pip install tabulate
pip install pandas
pip install apache-airflow-providers-slack
"""

from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.dummy import DummyOperator
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator

from pendulum import yesterday

from googleapiclient import discovery
from oauth2client.client import GoogleCredentials

import pandas as pd

class auto_firewall_check:
    def __init__(self):
        pass

    # GCP All-Open 방화벽 체크 메서드
    def firewall_checks(self):
            credentials = GoogleCredentials.get_application_default()

            service = discovery.build('compute', 'v1', credentials=credentials)

            # Project ID for this request.
            project = 'gcp-project-366909'
            # TODO: Update placeholder value.
            request = service.firewalls().list(project=project)
            lst = []
            while request is not None:
                response = request.execute()
                for firewall in response['items']:
                    # TODO: Change code below to process each `firewall` resource
                    sourceRanges = firewall['sourceRanges']
                    # all open 방화벽 여부 체크
                    if '0.0.0.0/0' not in sourceRanges:
                        continue
                    # json 형태로 만들기
                    firewall_id = firewall['id']
                    firewall_name = firewall['name']
                    firewall_network = firewall['network'].split('/')[-1]
                    traffic_direction = firewall['direction']
                    creation_date = firewall['creationTimestamp']
                    dic = {}
                    dic['firewall_id'] = firewall_id
                    dic['firewall_name'] = firewall_name
                    dic['firewall_network'] = firewall_network
                    dic['traffic_direction'] = traffic_direction
                    dic['creation_date'] = creation_date
                    lst.append(dic)
                request = service.firewalls().list_next(previous_request=request, previous_response=response)
            df = pd.DataFrame(lst)
            return df

    # exception이 작동하여 강제로 dag 실패시키기 위한 더미 메서드
    def exception_method(self, **context):
        raise AirflowException("Exception Happen")

    # failure callback
    def on_failure_callback(self, context):
        # 전송할 메세지
        slack_msg = """
            :red_circle: Task Failed.
            *Task*: {task}
            *Dag*: {dag}
            *Execution Time*: {exec_date}
            *Log Url*: {log_url}
            """.format(
                task=context.get('task_instance').task_id,
                dag=context.get('task_instance').dag_id,
                ti=context.get('task_instance'),
                exec_date=context.get('execution_date'),
                log_url=context.get('task_instance').log_url,
            )
        # SlackWebhookOperator 사용하여 http 통신
        failed_alert = SlackWebhookOperator(
            task_id='slack_notification_failure',
            http_conn_id='slack_webhook',
            message=slack_msg)
        return failed_alert.execute(context=context)

    # success callback
    def on_success_callback(self, context):
        # df 형태로 all-open 방화벽 가져오기
        df = self.firewall_checks()
        # 전송할 메세지
        message_result = ("GCP 인스턴스 방화벽에 0.0.0.0/0 으로 오픈 된 방화벽 정책이 있습니다.\n\n"
                          + "```"
                          + df.to_markdown()
                          + "```"
                          + "\n")
        slack_message = ":bell:" + " *방화벽 모니터링* \n" + message_result
        # SlackWebhookOperator 사용하여 http 통신
        success_alert = SlackWebhookOperator(
            task_id='slack_notification_success',
            http_conn_id='slack_webhook',
            message=slack_message
        )
        return success_alert.execute(context=context)
```

# 결과 출력

아래의 스크린샷은 t2 테스크를 사용해서 강제로 dag 실패를 유도한 뒤 에러메세지를 받은 것 입니다.

t1 테스크가 성공해서 방화벽 모니터링 메세지가 왔고 바로 이어서 t2 테스크가 실패해서 에러메세지가 온것을 확인 할 수 있습니다.

![auto-firewall-check9](/images/auto-firewall-check9.png)

동시에 dag가 실패했기때문에 에러메세지가 이메일로도 오게 됩니다.

![auto-firewall-check10](/images/auto-firewall-check10.png)

또한 airflow webserver에서 dag 성공을 확인할 수 있습니다.

![auto-firewall-check11](/images/auto-firewall-check11.png)

## 별첨

슬랙의 에러메세지와 이메일의 메세지의 형태가 다른 이유는 슬랙 에러메세지는 context를 참조해서 커스텀 메세지를 전송하도록 했기 때문입니다.

기본적으로 callback 함수에는 각 테스크의 상태정보가 딕셔너리 타입으로 전달이 되기 때문에 이를 활용한 것입니다.

context의 인자들은 airflow/models/__init__.py의 get_template_context에서 확인할 수 있습니다.

```python
return {
    'dag': task.dag,
    'ds': ds,
    'next_ds': next_ds,
    'next_ds_nodash': next_ds_nodash,
    'prev_ds': prev_ds,
    'prev_ds_nodash': prev_ds_nodash,
    'ds_nodash': ds_nodash,
    'ts': ts,
    'ts_nodash': ts_nodash,
    'ts_nodash_with_tz': ts_nodash_with_tz,
    'yesterday_ds': yesterday_ds,
    'yesterday_ds_nodash': yesterday_ds_nodash,
    'tomorrow_ds': tomorrow_ds,
    'tomorrow_ds_nodash': tomorrow_ds_nodash,
    'END_DATE': ds,
    'end_date': ds,
    'dag_run': dag_run,
    'run_id': run_id,
    'execution_date': self.execution_date,
    'prev_execution_date': prev_execution_date,
    'next_execution_date': next_execution_date,
    'latest_date': ds,
    'macros': macros,
    'params': params,
    'tables': tables,
    'task': task,
    'task_instance': self,
    'ti': self,
    'task_instance_key_str': ti_key_str,
    'conf': configuration,
    'test_mode': self.test_mode,
    'var': {
        'value': VariableAccessor(),
        'json': VariableJsonAccessor()
    },
    'inlets': task.inlets,
    'outlets': task.outlets,
}
```

# Reference

[Installation — Airflow Documentation (apache.org)](https://airflow.apache.org/docs/apache-airflow/2.0.1/installation.html)

[Dependencies — Airflow Documentation (apache.org)](https://airflow.apache.org/docs/apache-airflow/stable/installation/dependencies.html)

[Email Configuration — Airflow Documentation (apache.org)](https://airflow.apache.org/docs/apache-airflow/stable/howto/email-config.html)

[send email from airflow(naiveskill.com)](https://naiveskill.com/send-email-from-airflow/)

[Airflow slack alert(naiveskill.com)](https://naiveskill.com/airflow-slack-alert/)

[Airflow - Slack으로 결과 전달하기 - Mk’s Blog (moons08.github.io)](https://moons08.github.io/programming/airflow-slack/)

[Airflow - 성공, 실패시 Slack 전송 작업 (tistory.com)](https://jaeyung1001.tistory.com/241)

[꿈 많은 사람의 이야기 (tistory.com)](https://lsjsj92.tistory.com/634)

[Manage Airflow DAG notifications](https://docs.astronomer.io/learn/error-notifications-in-airflow)

[Airflow - Success and failure notifications (linkedin.com)](https://www.linkedin.com/pulse/airflow-success-failure-notifications-bipin-patwardhan?trk=pulse-article_more-articles_related-content-card)

[python - Error in airflow 2 even though my task's have actually completed? ERROR - Could not serialize the XCom value into JSON - Stack Overflow](https://stackoverflow.com/questions/70367529/error-in-airflow-2-even-though-my-tasks-have-actually-completed-error-could)

[서비스 계정을 사용하여 워크로드 인증 Compute Engine 문서 (Google Cloud)](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances#changeserviceaccountandscopes)

[airflow.models.baseoperator — Airflow Documentation (apache.org)](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/baseoperator/index.html)

[Airflow - task에서 return한 값 사용하기 (XCom) (tistory.com)](https://it-sunny-333.tistory.com/160)
