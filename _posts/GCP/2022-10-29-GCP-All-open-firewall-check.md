---
title: GCP - All open된 GCP 프로젝트 방화벽 체크 및 슬랙 알림 설정
categories: [GCP]
---

# all open check

지난 GCP 방화벽 규칙에 대한 포스팅을 위해 검색을 하다가 재밌는걸 하나 발견했습니다.

모든 엑세스에 대해 오픈된 GCP 방화벽 규칙을 찾아주고 이를 슬랙 메세지로 전송해주는 코드가 바로 그것입니다.

저야 소소하게 가지고 놀던 개인 프로젝트니 문제가 생겨도 그냥 새로운 프로젝트를 만들면 그만인데, 만약 회사같이 더 많은 인스턴스가 있는 프로젝트를 관리하는 곳에서 저와 같은 일이 생겼다면 어땠을까 하는 생각이 불쑥 들었습니다.

그러한 상황속에서 그 코드가 상당히 매력적이라고 느껴졌고, 그래서 저도 한번 따라만들고 싶어졌습니다.

근데 그 블로그에는 코드만 올라와있어서 되게 간단하게 끝날줄 알았는데 생각보다 앞서서 해야될 일들이 있어서 좀 헤맸습니다. 그리고 개인적인 욕심으로는 좀 자동화되면 좋겠다싶었습니다. 주기적으로 확인메세지를 받는게 관리가 더 편하니까요.

어쨌든 이것들을 진행하는 모든 과정들을 종합적으로 정리해보고 싶어서 나름 간단하게 기획안 짜고 이것저것 해보려합니다. 만약 완성하면 나중에 베리에이션도 다양하게 줄 수 있어서 상당히 괜찮을 듯하구요.

---

# ✅ ALL-OPEN 방화벽 체크 자동화 in GCP

## ⚒️사용 툴

**IDE**

- Pycharm

**구현**

- Python3 : v 3.7
- Google-api-python-client : v 2.65
- oauth2client : v 4.1.3
- pandas: v 1.3.5
- tabulate: v 0.9.0
- Gcloud CLI : v 405.0.0

**자동화**

- Airflow
- GCP Composer

**버전관리**

- Git/Github

## 💡 아이디어

- all-open 방화벽은 해킹에 상당히 취약함.
- 하나의 인스턴스만 뚫려도 GCP 보안 규정상 **전체 프로젝트가 블락**걸릴 수 있음
- 이때문에 정기적인 all-open 방화벽의 존재를 확인할 필요가 존재
- 프로젝트의 규모가 커질 수록 그 필요는 더 증가함

## 📝 계획

- Python 코드로 all-open 방화벽 조회 및 Slack 메세지 전송
- Airflow로 스케쥴링하여 정기적인 all-open 방화벽 체크
- GCP Composer를 사용해 정기적인 all-open 방화벽 체크
- Git과 Github으로 Airflow DAG 버전관리

---

# All-open 방화벽 체크

*본 포스팅은 **개인 windows OS 로컬 환경**에서 IDE(Pycharm)를 사용하여 python 코드로 사용자의 특정 GCP 프로젝트에 속한 인스턴스들의 방화벽 설정을 체크하기 위한 과정입니다.*

## 기초 세팅

우선 제일 먼저해야하는건 Compute Engine API를 Enable상태로 만드는겁니다. 기존에 VM을 만든 적이 있다면, 당연히 Enable 상태입니다.

API가 정상적으로 사용설정되었는지 확인하고 싶다면 [여기](https://console.developers.google.com/apis/api/compute)를 누르면 됩니다. 아래와 같이 상태에 사용설정됨이라고 뜨면 정상입니다.

![firewall-check1](/images/firewall-check1.png)

그 다음에는 인증 설정을 해야합니다. 이를 위해선 우선적으로 Google Cloud SDK 중 하나인 Google Cloud CLI(gcloud CLI)를 설치해줘야합니다.

설치는 [관련 페이지](https://cloud.google.com/sdk/docs/install-sdk)를 보면 됩니다.

![firewall-check2](/images/firewall-check2.png)

windows버전은 그냥 설치 프로그램 받아서 실행시키기만 하면됩니다.

바탕화면에 바로가기를 생성하고, 이를 실행시켜주면 터미널이 하나 뜹니다. 이것이 Google Cloud CLI입니다.

![firewall-check3](/images/firewall-check3.png)

이제 애플리케이션 기본 사용자 인증 정보(ADC)를 설정해줘야합니다.

ADC는 애플리케이션 환경을 기준으로 사용자 인증 정보를 자동으로 검색하고 이러한 사용자 인증 정보를 사용해서 Google Cloud API에 인증하기 위해 Cloud 클라이언트 라이브러리 및 Google API 클라이언트 라이브러리에 사용되는 전략입니다.

ADC를 설정하고 클라이언트 라이브러리를 사용할 때는 Google Cloud 서비스 및 API에 대한 애플리케이션 인증 방법을 변경하지 않고 개발 또는 프로덕션 환경에서 코드를 실행할 수 있습니다.

이번 포스팅에서는 코드 작성 때, [방화벽 리스트를 출력하는 API](https://cloud.google.com/compute/docs/reference/rest/v1/firewalls/list?hl=id-ID)를 사용하기 때문에 ADC가 필요한 것입니다.

[공식홈페이지](https://cloud.google.com/docs/authentication/provide-credentials-adc)에는 여러 환경에 따라 ADC 설정하는 방법이 잘 설명되어있습니다. 전 앞서 말했듯 로컬에서 실행할 것이기 때문에 아래 명령어를 CLI에 입력해줍니다.

```bash
# In Google Cloud CLI

gcloud beta auth application-default login
```

명령어치면 터미널 창이 하나 더 열리고 beta가 설치될겁니다.

설치가 끝나면 다시 원래 gcloud CLI로 돌아와서 위의 커맨드를 다시 입력해줘야합니다.

이젠 웹브라우저가 켜지고 구글 계정 선택 페이지가 보일겁니다. **GCP에 가입한 계정 클릭 → 허용** 하고 아래와 같은 창이 뜨면 완료된겁니다,

![firewall-check4](/images/firewall-check4.png)

그리고 다시 gcloud CLI로 가면, 아래와 같은 메세지들이 출력되어 있을 겁니다.

![firewall-check5](/images/firewall-check5.png)

Credentials saved to file은 인증파일(application_default_credentials.json)이 저장되어있는 경로를 의미하합니다. Google cloud libraries는 해당 경로에 위치한 json 파일을 읽고 접근을 허용하게 됩니다.

Quota project "프로젝트명" was added to ADC … 은 프로젝트가 ADC에 추가되었음을 알리는 메세지입니다. 여기서 중요한건 큰따음표(”) 사이에있는 프로젝트 명을 별도로 저장해둬야한다는 점입니다.

아마 자신의 프로젝트명 + 숫자로 된 형태일텐데 이후 코드에서 프로젝트명을 적을 때, 이것을 사용해야기 때문입니다.

## 방화벽 리스트 체크 코드 세팅

우선 두개의 라이브러리를 설치해 줄겁니다.

```python
# 구글 API 호출하기 위한 파이썬 라이브러리
pip install google-api-python-client

# ADC 인증을 위한 라이브러리
pip install oauth2client
```

GCP는 [방화벽 리스트를 출력하는 API와 그 예시](https://cloud.google.com/compute/docs/reference/rest/v1/firewalls/list?hl=id-ID)를 제공해주기 때문에 이를 기반을 작성할 겁니다. 밑의 코드가 바로 그것입니다.

```python
"""
BEFORE RUNNING:
---------------
1. If not already done, enable the Compute Engine API
   and check the quota for your project at
   https://console.developers.google.com/apis/api/compute
2. This sample uses Application Default Credentials for authentication.
   If not already done, install the gcloud CLI from
   https://cloud.google.com/sdk and run
   `gcloud beta auth application-default login`.
   For more information, see
   https://developers.google.com/identity/protocols/application-default-credentials
3. Install the Python client library for Google APIs by running
   `pip install --upgrade google-api-python-client`
"""
from pprint import pprint

from googleapiclient import discovery
from oauth2client.client import GoogleCredentials

credentials = GoogleCredentials.get_application_default()

service = discovery.build('compute', 'v1', credentials=credentials)

# Project ID for this request.
project = 'my-project'  # TODO: Update placeholder value.

request = service.firewalls().list(project=project)
while request is not None:
    response = request.execute()

    for firewall in response['items']:
        # TODO: Change code below to process each `firewall` resource:
        pprint(firewall)

    request = service.firewalls().list_next(previous_request=request, previous_response=response)
```

결과는 Json으로 가져옵니다.

```python
{'allowed': [{'IPProtocol': 'tcp', 'ports': ['80']}],
 'creationTimestamp': '2022-10-28T04:41:53.238-07:00',
 'description': '',
 'direction': 'INGRESS',
 'disabled': False,
 'id': '-',
 'kind': 'compute#firewall',
 'logConfig': {'enable': False},
 'name': 'default-allow-http',
 'network': '-',
 'priority': 1000,
 'selfLink': '-',
 'sourceRanges': ['0.0.0.0/0'],
 'targetTags': ['http-server']}
```

sourceRanges를 보면 해당 방화벽이 본 포스팅에서 타겟하는 all-open 방화벽임을 알 수 있습니다. 따라서 위의 코드에서 sourceRanges에 0.0.0.0/0인 경우에 대하여 조건을 걸어주면 됩니다.

## 방화벽 체크 코드

all-open 방화벽을 걸러내고, all-open 방화벽의 아이디, 이름, 사용중인 VPC 네트워크, 트래픽 방향, 생성일자를 추출해서 json형태로 가공하였습니다.

```python
from pprint import pprint

from googleapiclient import discovery
from oauth2client.client import GoogleCredentials

credentials = GoogleCredentials.get_application_default()

service = discovery.build('compute', 'v1', credentials=credentials)

# Project ID for this request.
project = '자신의 프로젝트명을 입력하세요'  # TODO: Update placeholder value.

request = service.firewalls().list(project=project)
lst = []
while request is not None:
    response = request.execute()

    for firewall in response['items']:
        # TODO: Change code below to process each `firewall` resource:
        sourceRanges = firewall['sourceRanges']
        if '0.0.0.0/0' not in sourceRanges:
            continue

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

pprint(lst)
```

## 최종 코드(방화벽 리스트 체크 + 슬랙 전송)

특이사항으로는 pandas의 to_markdown 메서드를 사용하기 위해서 tabulate를 설치 해줘야합니다.

pandas의 to_markdown 메서드를 사용하면 데이터프레임을 마크다운형태로 출력한 값을 객체에 담을 수 있습니다.

이를 통해 슬렉 메세지가 자동으로 깔끔한 데이터프레임 형태를 그려낼 수 있도록 했습니다.

```python
pip install tabulate
```

슬랙설정하는 법은 [별도의 포스팅](https://won21yuk.github.io/posts/%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9C%BC%EB%A1%9C-%EC%8A%AC%EB%9E%99-%EB%A9%94%EC%84%B8%EC%A7%80-%EB%B3%B4%EB%82%B4%EA%B8%B0/)으로 정리해두었습니다.

그렇게 완성된 최종 코드입니다.

```python
from pprint import pprint

from googleapiclient import discovery
from oauth2client.client import GoogleCredentials

import requests, json
import pandas as pd

credentials = GoogleCredentials.get_application_default()

service = discovery.build('compute', 'v1', credentials=credentials)

# Project ID for this request.
project = '자신의 프로젝트명을 입력하세요'  # TODO: Update placeholder value.

request = service.firewalls().list(project=project)
lst = []
while request is not None:
    response = request.execute()

    for firewall in response['items']:
        # TODO: Change code below to process each `firewall` resource:
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

# 데이터 프레임 만들기
df = pd.DataFrame(lst)

# 슬랙에 연결 및 사용할 메세지 세팅
slack_webhook_url = "자신의 webhook URL 입력하세요"

# 슬랙 메세지는 마크다운 언어를 지원하기 때문에 마크다운 언어로 구성되도록 작성해도 됨.
message_result = ("GCP 인스턴스 방화벽에 0.0.0.0/0 으로 오픈 된 방화벽 정책이 있습니다.\n\n"
                  + "```"
                  + df.to_markdown()
                  + "```"
                  + "\n")
slack_message = ":bell:" + " *방화벽 모니터링* \n" + message_result

# 슬랙에 메세지 보내기
payload = {"text": slack_message,
           "icon_emoji": "false"}
requests.post(slack_webhook_url,
              data=json.dumps(payload),
              headers={"Content-Type": "application/json"})
```

![firewall-check6](/images/firewall-check6.png)

다음 포스팅에서는 자동화 작업을 진행하겠습니다.

# Reference

[Method: firewalls.list | Compute Engine Documentation | Google Cloud](https://cloud.google.com/compute/docs/reference/rest/v1/firewalls/list?hl=id-ID)

[빠른 시작: Google Cloud CLI 설치 | Google Cloud CLI 문서](https://cloud.google.com/sdk/docs/install-sdk)

[GCP - all open 방화벽 체크 (tistory.com)](https://burning-dba.tistory.com/126?category=1027247)

[python - Send a pandas dataframe to slack - Stack Overflow](https://stackoverflow.com/questions/63781437/send-a-pandas-dataframe-to-slack)
