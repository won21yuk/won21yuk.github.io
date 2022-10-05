---
title: Hadoop Deep Inside - Ch.0 개요
categories: [Hadoop, Deep Inside]
---

![하둡](/images/hadoop.jpg "Hadoop")
> Hadoop Deep Inside 시리즈는 T-Academy의 하둡강의를 기반으로 진행합니다.   \
> 이 외에도 하둡 공식홈페이지와 여러 블로그, 커뮤니티들을 참조합니다.   \
> 또한, Deep Inside 시리즈는 spark, elk stack, kafka 등 사용한 경험이 있는 모든 소프트웨어를 대상으로 확대해 갈 예정입니다.

# 0. 여는 말
빅데이터를 공부하는 사람이라면, 하둡이라는 소프트웨어는 한번 쯤은 들어 보셨겠죠. 특히, 저처럼 데이터 엔지니어를 꿈꾸는 사람들에게는 나름 더 친숙할 겁니다. 데이터 파이프라인을 구축할 때 반드시 포함되는 소프트웨어 중 하나이기 때문입니다.   \
당장 저같은 경우만 봐도 멀티캠퍼스의 데이터 엔지니어 과정 중에 세 번의 프로젝트를 진행하면서 하둡을 다뤄봤고, 개인적으로는 GCP 환경에서 고가용성 하둡 클러스터를 구축해본 경험이 있으니까요.   \
그럼에도 현시점의 저는, 누군가 하둡이 뭔지 말해 보라고 한다면 '_데이터 분산 저장 및 처리하는 소프트웨어야..._'라며 대충 넘길 수준에 불과합니다. 이는 제가 무엇을 하는 지도 모른채 맹목적으로 사용하는데 급급했었기 때문입니다.   \
 그런 점에서 저는 '처음이니까, 아직 시작한지 얼마 안됐으니까, 지금 프로젝트를 마치는게 더 중요하니까'라며 중요한 숙제들을 미루고 있었던걸지도 모르겠습니다.
 그래서 제가 하둡을 싱글노드로 구축했던지, 클러스터로 구축했던지, 아니면 더 나아가 고가용성 클러스터를 구축하였던지 그런 것들은 별로 중요하지 않게되었습니다. 그런 경험보단, 하나를 하더라도 제가 올바르게 이해하고 사용했는지가 더 중요하다는 생각이 들어버렸으니까요.   \
 저는 이제부터 Hadoop Deep Inside라는 시리즈를 통해 이제껏 미룬 숙제를 해보려합니다. 부디 제가 이 시리즈를 하나씩 끝맺을 때마다 '난 이 소프트웨어를 안다'라고 당당하게 말할 수 있는 사람이 되었으면 좋겠습니다.

* * *

# 1. 왜 하둡인가?
![why-hadoop](/images/Why-Hadoop.jpg "why-hadoop?")
다양한 무료 강의들을 제공해주는 DataFlair라는 사이트에서 하둡을 배워야하는 핵심 이유 11가지를 선정   \
이하 내용은 [DataFlair](https://data-flair.training/blogs/why-hadoop/)을 번역하여 요약한 내용

## 1) 빅데이터 관리(Managing Big Data)
![Data-Distribution](/images/Growth-of-Unstructured-Data.jpg "Growth-of-Unstructured-Data")
우리는 데이터 홍수의 시대(2일마다 5엑사바이트의 데이터 생산)에 살고 있으며, 기하급수적으로 증가하는 데이터 양을 관리하기 위한 빅데이터 기술이 필요.
강력한 아키텍쳐와 경제적인 기능을 가진 하둡은 방대한 데이터를 저장 및 처리하는데 가장 적합한 도구.

## 2) 빅데이터 시장의 기하급수적인 성장(Exponential Growth of Big Data Market)
_Forbes - "하둡 시장은 연 평균 42.1%로 2022년까지 99.31억 달러에 다를 것으로 예측."_
![Hadoop market](/images/Global-Hadoop-Market.jpg "Global-Hadoop-Market")
빅데이터 시장이 점점 성장함에 따라 빅데이터 기술에 대한 욕구도 증가. 하둡은 이렇게 생겨난 빅데이터 기술(spark, flink 등)들의 토대를 형성.
이러한 이유들로 하둡 전문가에 대한 수요가 점차 증가하고 있음.

## 3) 하둡 전문가의 부족(Lack of Hadoop Professionals)
하둡 전문가들에 대한 수요는 늘어나고 있지만, 여전히 하둡 전문가의 수는 부족.
하둡이 기본적으로 소프트웨어로 구성되어 있는 플랫폼인데 단순히 소프트웨어를 안다고 하둡을 잘할 수 있는게 아니라 TA(Technical Architect)


## 4) 모두를 위한 하둡(Hadoop for all)

## 5) 강력한 하둡 에코시스템(Robust Hadoop Ecosystem)

## 6) 분석 도구(Research Tool)

## 7) 사용 편의성(Ease of Use)

## 8) 하둡은 어디에나 있다(Hadoop is Omnipresent)

## 9) 더 높은 급여(Higher Salaries)

## 10) 성숙하는 기술(A Maturing Technology)

## 11) 더 나은 커리어 범위(Hadoop has a Better Career Scope)

# 2. 하둡의 역사

