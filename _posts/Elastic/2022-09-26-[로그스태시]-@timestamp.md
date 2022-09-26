---
title: 로그스태시 1. @timestamp
categories: [Elastic, Logstash]
---

> elasticsearch : v 8.4.1
> logstash : v 8.4.1
> kibana : v 8.4.1

Kafka topic에서 가져온 Log data를 Elasticsearch에 넣는 과정에서 @timestamp가 UTC 기준으로 되어있다는 것을 알아챘다.
로그 스태시는 default로 UTC 시간을 @timestamp에 표시하게 되어있는데 한국 시간 대비 9시간 전 시간이 나타난다.
나는 Kibana에서 로그데이터의 시계열 분석을 계획했었기 때문에, 로그가 찍힌 시간이 올바르지 못하면 제대로된 결과값을 얻기 힘들것이라 판단했다.

### 1. 첫번째 시도
 첫 번째로 시도했던 건 단순히 9시간을 더 해주는 것 이였다. 로그스태시의 data filter로 @timestamp에 0900값을 더해 주었다.
시간은 9시간이 더해진 값으로 출력이 됐지만 문제는 date type 였던 @timestamp 필드가 string 필드로 바뀐다는 것이다.
시계열로 로그데이터를 시각화해야하는 입장에서 용인할 수 없는 부분이여서 다른 방법이 있는지를 찾아봤다.

### 2. 두번째 시도
 두 번째로 찾은 방법은 date filter의 인자로 timezone의 값을 Asia/Seoul로 주는 것이였다.
하지만 표시된 시간은 여전히 UTC로 나타나고 Elasticsearch에도 UTC의 값 그대로 색인되고 있었다.

### 3. 세번째 시도
 내가 @timestamp를 건든 이유는 kibana에서 시각화하여 시계열로 로그데이터를 분석하기 위함이다.
내 목적에만 충실히 집중했을때, kibana에서 @timestamp가 KTC로 표시만 된다면 Elasticsearch에 UTC로 찍히는 것은 아무런 문제가 되지 않는다.
그래서 kibana 설정에서 timezone의 값을 Asia/Seoul로 바꿔주었더니 kibana로 시각화 할때는 @timestamp의 값이 저절로 9시간이 더해진 값 즉 KCT로 표시되는 것을 확인 할 수 있엇다.


사실 엄청 간단한 문제일 것이라 생각 했는데 logstash 상에서 UTC를 KTC로 바꾸는 방법은 찾지 못했다.
kibana의 timeznoe 값을 바꾸는 것도 사실 일찍이 찾았지만 임시 방편이라고 생각해 뒤로 미뤄두었는데
logstash 상에서 KCT로 바꾸는 법을 못 찾았기 때문에 불가피하게 Kibana의 설정을 건들 수 밖에 없었다.

