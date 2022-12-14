---
title: Docker - 왜 도커는 이렇게 인기가 많을까?
categories: [Docker, Docker Notion]
---

도커가 인기가 많다는 건 부정할 수 없습니다. 그런데 제가 궁금한건 신기술도 아닌데 왜 사람들이 도커에 열광했을까하는 부분입니다. 도커가 처음 등장한 2013년에도 이미 컨테이너 기술은 존재했었습니다. 심지어 도커도 기존에 있던 컨테이너 기술 중 하나인 LXC를 기반으로 만든 것에 불과합니다. 그럼에도 도커는 엄청난 인기를 얻었습니다. 이제는 개발자의 당연한 소양처럼 되어버렸고 많은 기업들도 도커를 기반으로 자신들의 개발환경이나 인프라를 마이크로서비스 아키텍처로 만드는 상황에 도달했습니다. 그럼 도커는 이전의 컨테이너와 무엇이 달랐고 무엇이 뛰어났길래 이렇게까지 인기가 많아진걸까요? 이 근본적인 의문을 지금부터 해소해보도록 하겠습니다.

# 게임 체인저

컨테이너라는 개념 자체는 1960년대 처음 등장했습니다. 이 개념의 구현체가 처음 등장한 것을 시작점으로 잡는다면 2000년에 나온 FreeBSD의 Jail이라고 볼 수도 있습니다. 적어도 20년은 넘은 개념인 것이죠. 그 긴 역사만큼이나 LXC, jail, solaris zones, OpenVZ, Imctfy같은 다양한 컨테이너 기술들이 등장했지만 대중화되는데는 실패했습니다. 반면 도커는 2013년에 등장한 비교적 젊은 기술입니다. 그러나 흥미롭게도 지난 몇년동안 엄청난 인기를 얻은건 더 성숙한 기술들이 아닌 바로 도커였습니다.

![why-is-docker-popular0](/images/why-is-docker-popular0.png)

도커가 왜 인기가 많아졌느냐에 대한 이야기들은 분분합니다. 주관적인 영역이기에 모든 내용들이 통일되긴 어려운게 사실입니다. 다만 대체로 공감하는 이야기가 하나 있다면 도커는 시의적절하게 등장했다는 점입니다.

도커는 처음 등장할 때부터 오픈소스로 등장했습니다. 따라서 도커는 오픈 소스를 소프트웨어 생산의 기본으로 보기 시작한 기술 시장에서 어필할 수 있었습니다. 혹자는 말하길 도커가 5년전에 등장했다면 지금과 같은 강렬한 인기를 끌지 못했을 것이라 말하기도 합니다.

또한 당시 대부분의 기업은 가상머신을 활용하여 어플리케이션을 만들고 배포하는 체계가 무르익은 상태였습니다. 기업들은 무겁고 느린 가상머신으로 인해 수년간 어플리케이션을 배포하는 더 간결한 방법을 찾고 있었고 때마침 등장한 것이 도커입니다. 앞선 컨테이너 기술인 OpenVZ나 LXC가 등장했을 때에도 비슷한 솔루션을 제공했지만 전통적인 가상화는 성숙하지 않았기 때문에 이러한 컨테이너 기술들은 덜 매력적이였을 겁니다.

그리고 도커는 2010년대 초반 일어난 DevOps 혁명의 가치와 궤를 같이했습니다. DevOps는 소프트웨어 제공에 있어 신속함, 유연성, 확장성을 강조합니다. 도커 역시 빠르고 유연하게 어플리케이션 배포를 할 수 있고 이미지를 통해 손쉽게 확장할 수 있습니다. 이때문에 도커 컨테이너는 DevOps의 파트너로써 각광받았습니다.



결국 이러한 시기적인 이유들이 겹치면서 도커는 전통적인 가상화를 대체하는 소위 게임체인저로 필드에 자리잡게 된 것입니다. 물론 이 이야기들은 도커가 등장했을 당시에 초점을 두고 있기 때문에 도커가 지속적으로 사랑받는 이유에 대해선 설명하지 못합니다. 그래서 다음 단락에서 왜 도커가 지금까지도 인기가 많은지에 대해서도 이야기해보려 합니다.

# 도커가 인기가 많은 5가지 이유

시의적절하다는건 한가지 운적인 요소에 불과할 겁니다. 실제로 도커를 사용하는데 명확한 이점들이 있으니 잠깐의 관심에 그치지 않고 사람들이 꾸준히 도커를 사용했을 것이니까요. 그리고 그 이유들은 비교적 뚜렷하게 찾아볼 수 있었습니다. 많은 이유들이 존재하지만 이번 단락에서는 많은 사람들이 공통적으로 이야기하는 것들만 5가지를 추려서 살펴보도록 하죠.

## 마이크로서비스 아키텍처(MSA)

![why-is-docker-popular1](/images/why-is-docker-popular1.png)

오늘날에는 모놀리식 어플리케이션에서 벗어나 여러 구성요소로 빌드된 어플리케이션이 있는 마이크로서비스 아키텍처로 향하는 추세입니다. 컨테이너를 사용하면 마이크로서비스 아키텍처로 쉽게 이전할 수 있고 그 과정에서 발생하는 마찰도 적습니다.

이처럼 마이크로서비스 아키텍처와 컨테이너는 상호의존적입니다. 컨테이너는 마이크로 서비스를 수행할 수 있는 기능을 제공하고 마이크로서비스는 컨테이너와 함께 잘 작동합니다. 또한 개발자는 마이크로서비스를 빌드하는 것이 편리하기 때문에 컨테이너를 좋아하고 마이크로서비스가 인기가 있는 이유 중 한가지도 컨테이너를 통해 쉽게 마이크로서비스를 구현할 수 있기 때문입니다.

## 호환성

서버 사양이 달라서, 커널 버전이 달라서, 설치된 라이브러리가 맞지 않아서 등 정상적으로 작동하던 어플리케이션이 다른 곳에서 제대로 작동하지 않는 이유는 너무 많았습니다.

하지만 도커 컨테니어는 가상 머신과는 달리 호환성 문제없이 모든 플랫폼에 배포할 수 있습니다. 개인 컴퓨터든 클라우드든 배포 환경과는 상관이 없습니다. 어플리케이션은 시스템에 구애받지 않으므로 모든 호스트 시스템 또는 클라우드에 더 쉽게 사용, 구축, 관리, 배포 할 수 있습니다. 이로써 개발자들은 운영환경과 관계없이 어플리케이션을 실행할 자유를 얻었고 패키징한 어플리케이션의 실행 독립성이 보장됐다고 말할 수 있게 되었습니다.

특히나 최근에는 멀티 클라우드 환경이 인기를 끌고 있습니다. 멀티 클라우드 환경에서 각 클라우드는 서로 다른 구성, 정책 및 프로세스로 제공되며 Terraform, Ansible, Chef, Puppet 등 서로 다른 인프라 관리 툴을 사용하여 관리됩니다. 그러나 도커 컨테이너는 모든 환경으로 이동할 수 있습니다. 가령 AWS EC2 인스턴스에서 실행되는 컨테이너를 Google Cloud Platform 환경으로 원활하게 이동시킬 수 있습니다.

## 효율적인 비용

도커는 가상머신과 달리 호스트가 직접 리소스를 할당하는 가상화의 한 형태입니다. 이렇게하면 가상 머신을 띄우는 것보다 훨씬 많은 수의 도커 컨테이너를 실행할 수 있습니다. 각 컨테이너 자체는 어플리케이션의 필요에 따라 리소스를 할당하기 때문에 효율적입니다.

또한 도커는 계층화된 파일 시스템을 사용합니다. 이를 통해 도커는 파일을 효율적으로 재사용할 수 있으므로 디스크 공간을 덜 사용합니다. 가령 동일한 기본 이미지를 사용하는 여러 도커 컨테이너가 있는 경우에 도커는 필요한 파일의 단일 복사본만 유지하고 각 컨테이너와 공유합니다. 이는 방대한 규모의 경제를 창출하는 것과 같고 어플리케이션을 비용측면에서 효율적이게 합니다.

이와같은 도커의 비용절약적인 특성은 인프라를 구축하는데 필요한 리소스의 절감을 촉진할 수 있다는 것을 의미합니다. 인프라 요구 사항이 감소했기 때문에 기업은 서버 비용에서 유지관리에 필요한 직원에 이르기 까지 모든 것을 절약할 수 있습니다. 간단히 말하면 도커로 인프라를 구축하면 ROI가 높습니다.

## 속도

도커 컨테이너는 OS를 부팅할 필요가 없기 때문에 몇초만에 생성하고 배포할 수 있습니다. 그리고 그만큼 쉽게 컨테이너를 중단시키거나 파괴시킬 수도 있습니다. YAML 파일(도커 컴포즈 파일)을 작성하여 구성파일을 생성하면 손쉽게 구축을 자동화하고 인프라를 확장할 수도 있습니다.

도커를 사용하면 컨테이너 이미지를 생성하여 파이프라인 전체에서 사용하면서 종속되지 않는 작업을 병렬로 실행할 수 있기때문에 CI/CD 파이프라인의 속도와 효율성이 향상됩니다. 이를 통해 어플리케이션의 개발기간을 단축시킬 수 있고 생산성도 크게 향상됩니다. 만약 변경사항을 커밋하고 버전을 제어하는 도커 이미지를 사용하면 새로운 변경사항이 발생할 경우 즉시 이전 버전으로 롤백할 수 도 있습니다.

## 보안

![why-is-docker-popular2](/images/why-is-docker-popular2.png)

도커 환경은 매우 안전합니다. 컨테이너들은 서로 완전히 격리되어 권한이 부여된 엑세스 없이 다른 컨테이너의 데이터에 엑세스 할 수 없습니다. 원하는 경우에는 보안 계층을 추가하여 보안을 강화시킬 수도 있습니다.

# 도커의 인기는 어느정도인가

이 단락은 그냥 진짜 얼마나 인기가 있는걸까 하는 순수한 호기심에 찾아본 결과입니다. 그냥 막연히 인기가 많다는 것보단 확연한 수치가 있는편이 납득하기 좋으니까요. 그래서 관련 자료를 찾다가 발견한 것이 Stackoverflow에서 해마다 발표하는 개발자 설문입니다.

- 개발자들이 가장 사랑하는 도구

![why-is-docker-popular3](/images/why-is-docker-popular3.png)

- 해당 언어 또는 기술로 개발하지는 않지만 해당 언어 또는 기술을 사용하여 개발하는 데 관심을 표명한 개발자의 비율

![why-is-docker-popular4](/images/why-is-docker-popular4.png)

[위의 자료](https://survey.stackoverflow.co/2022/)는 Stackoverflow에서 59,164명의 개발자를 대상으로 설문하여 2022년 5월에 발표한 따끈따끈한 자료입니다. 도커는 가장 사랑받는 그리고 가장 배우길 원하는 도구로써 당당히 1위에 선정되어 있습니다. 쿠버네티스도 한때 도커를 기반으로 했었던 컨테이너 오케스트레이션 도구기 때문에 컨테이너 관련한 기술 두 개가 1,2등을 나눠가진 형상입니다.

이외의 특징으로는 IaC가 꽤 핫한 것으로 보입니다. IaC 툴 중 Terrform, Ansible, Puppet, Chef까지 총 4개가 가장 사랑받는 그리고 가장 배우길 원하는 도구 TOP 13에 들어 있습니다.

# 마무리

도커가 왜 인기가 생겼는지 그리고 왜 지금까지도 이렇게 개발자들에게 사랑받는지에 대해서는 알 수 있었습니다. 왠지 도커에 대한 이야기가 길어지는데 그래도 꼬리에 꼬리를 무는 의문이 생긴다는건 뭔가 제대로 공부하고 있는 걸지도 모른다는 알량한 성취감을 줍니다. 그럼에도 저의 최종목적지는 쿠버네티스 그리고 그 너머에 있는 것이기 때문에 한동안은 부지런히 포스팅을 작성해야할듯합니다.
