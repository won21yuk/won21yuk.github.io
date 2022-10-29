# python으로 Slack에 메세지 보내기

외부환경에서 slack에 메세지를 보내기 위한 방법은 크게 두가지가 있습니다.

http통신을 하거나 slack SDK를 사용하여 통신하는 방법이 바로 그것입니다.

둘 중 무엇을 선택하든지 외부환경에서 슬랙에 메세지를 게시하려면, 기본적으로 Incoming webhook이라는 것을 사용해야합니다. 그리고 Incoming webhook은 슬랙에서 제공하는 앱을 통해 메세지를 게시하는 가장 간단한 방법입니다. 

## Incoming Webhook 시작하기

Slack App이 아직 없는 경우를 산정하고 진행해보겠습니다.

기본적으로 App을 만들어야 webhooks를 만들 수 있고, 이 webhook는 App당 하나씩 할당됩니다.

![Untitled.png](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled.png)

[슬랙 홈페이지](https://api.slack.com/)에가면 위와 같은 화면이 뜨는데 여기서 Create an app을 클릭하시면 됩니다.

![Untitled.png](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%201.png)

여기선 From scratch 설정을 사용합니다. 기본설정의 app으로 만들겠다는 의미입니다. 아래는 menifest라는 설치 구성정보를 담고 있는 json 설정 파일로 app을 만들때 사용하는겁니다. 아마 일반적인 사용자라면 사용할일이 거의 없을듯 합니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%202.png)

이젠 App 이름과 어느 워크스페이스에서 사용할건지 선택해주시면 됩니다. 기존의 워크스페이스가 아닌 다른 워크스페이스를 만들어서 사용하고 싶으시다면, 위의 그림 중하단부에 위치한 sign into a different workspace눌러서 새로운 워크스페이스를 만들면 됩니다.

![Untitled.png](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%203.png)

앱을 생성하고나면 Basic information 화면이 나올텐데 조금 스크롤내리면 위와같은 화면을 볼 수 있을겁니다. 여기서 Incoming Webhooks를 선택해줍니다.

설명에서 볼 수 있듯이 외부 소스로부터 슬랙으로 메세지를 게시할수 있게 해주는 기능입니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%204.png)

webhook은 디폴트로 off설정이 되어있기때문에 이를 on으로 바꿔주시면됩니다.

![Untitled.png](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%205.png)

그리고 밑으로 스크롤을 내려서 Add New Webhook to Workspace를 눌러서 Webhook이 작동중인 App과 워크스페이스 연결을 설정해주시면 됩니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%206.png)

webhook은 URL형태로 제공이됩니다. 외부환경에서 슬랙에 메세지를 보내기 위해 엑세스할때, 해당 URL을 통해 통신하기 위함입니다.

그래서 이는 이후 python 코드 작성할 때도 필요합니다. 따로 메모를 해놓아도 되고, 아니면 [여기](https://api.slack.com/apps/A048QN8RPSQ/incoming-webhooks?)에서 확인해도 됩니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%207.png)

위의 모든 작업을 마치고 연결했던 워크스페이스에 접속하면 앱이 추가되어있는 것을 확인할 수 있습니다.

## HTTP 통신

우선 간단하게 HTTP 통신을 하여 메세지를 보내보도록 하겠습니다. 

```python
import requests

def send_message(msg):
	url = "webhook URL" # 자신의 Webhook URL 입력
	data = {'text': msg}
	resp = requests.post(url=url, json=data)
	return resp

if __name__ == '__main__':
	send_message('HTTP 통신 테스트')
```

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%208.png)

## Slack SDK

SDK를 사용하면 다양한 라이브러리들을 사용할 수 있게되어 단순 HTTP통신 이상으로 더욱 풍부하고 간결한 작업이 가능해집니다. 다만, 토큰을 사용한 인증방식이 도입되고 라이브러리를 설치해줘야하는 등 추가적인 작업들이 발생합니다.

우선 [슬랙 API 홈페이지](https://api.slack.com/)에 들어가서 우측 상단에 있는 Your apps를 클릭해 본인이 사용하고자하는 app을 선택해주시면 됩니다.

저는 gcp-firewall-check-bot은 위에서 썼으니, Firewall Warning bot을 사용할 겁니다. 

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%209.png)

왼쪽 메뉴바에서 OAuth & Permissions를 클릭해줍니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2010.png)

여기 적힌 Token을 사용해서 SDK를 사용할 수 있습니다. 

![Untitled.png](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2011.png)

밑으로 스크롤을 좀 내리면 Scopes라는 설정이 보일겁니다. 여기서 chat:write를 선택하여 추가해줍니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2012.png)

그러면 상단에 노란색 bar가 생길텐데, 그때 reinstall your app을 클릭해줍니다.

![Untitled.png](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2013.png)

절차에 따라 진행하고, 다시 slack으로 돌아갑니다. 슬랙에서는 해당 app을 사용할 채널으로 이동하면됩니다. 이전 절차에서 선택했던 그 채널로 가시면 됩니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2014.png)

해당 채널에는 밑과 같은 메세지가 떠있을텐데 파란색 링크가 걸린 Firewall Warning bot을 눌러줍니다. 이건 자신의 설정한 app이름으로 되있을겁니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2015.png)

여기서 채널에 이앱추가를 눌러주면 추가되었다는 메시지가 뜨면 이제 완료된겁니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2016.png)

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2017.png)

이제 코드를 작성해보죠.

우선 slack-sdk를 설치해줍니다.

```python
pip install slack-sdk
```

그리고 간단하게 메세지 전송하는 코드를 작성해보겠습니다.

```python
from slack_sdk import WebClient

token = '토큰을 입력해주세요.'

client = WebClient(token=token)

response = client.chat_postMessage(channel='#일반', text='SDK 테스트')
```

이 코드를 실행시켜보면, 슬랙에 메세지가 정상적으로 작성된것을 확일할 수 있습니다.

![Untitled](python%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20Slack%E1%84%8B%E1%85%A6%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%87%E1%85%A9%E1%84%82%E1%85%A2%E1%84%80%E1%85%B5%20d31e88ac691546bbbe94058363ac75fd/Untitled%2018.png)