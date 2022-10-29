---
title: GCP - Google Cloud SDK
categories: [GCP]
---

# Google Cloud SDK

> Google Cloud **Software Development Kit**
>

![gcloud-sdk1](/images/gcloud-sdk1.png)

Google Cloud SDK는 구글 클라우드의 API들이나 서비스와 상호작용하기위해 사용되고, 사용자의 구글 클라우드 프로젝트들의 리소스를 관리하기 위해 사용되는 도구들의 집합입니다.

사용자는 이를 통해 다양한 공간에서 Google Cloud와 관련된 더 풍부한 작업들을 수행할 수 있게됩니다.

Google Cloud SDK의 구성요소는 크게 세가지로 구분할 수 있습니다.

우선 첫째, Google Cloud CLI입니다.

Google Cloud CLI는 커맨드 라인에서 클라우드에 접근하고 관리하기위해 [설치](https://cloud.google.com/sdk/docs/install-sdk)할 수 있는 커맨드 라인 도구모음입니다.

이 tool을 사용하면 Linux, Mac OS, Windows 터미널에서 가상머신(VM) 관리, Big Query 테이블 쿼리, Google Cloud Storage 버킷 업데이트 등 Cloud Console에서 할 수 있는 거의 모든 작업들을 수행가능합니다.

이는 운영 체제 패키지 관리자를 통해 사용하거나 cloud.google.com에서 직접 다운가능합니다.

둘째, Cloud Client Libraries입니다.

Cloud Client Libraries는 Google API를 코드에 프로그래밍 방식으로 통합하기 위한 Cloud Client Libraries입니다. pip나 npm같은 표준 라이브러리 패지키 매니저로 설치하여 사용이 가능합니다.

Cloud Client Libraries는 Java, Python, Node.js, GO 를 비롯한 여러 프로그래밍 언어에서 사용할 수 있습니다.

즉, 사용가능한 언어별 프레임워크와 도구가 있다는 겁니다. 따라서 선호하는 언어가 무엇이든 상관없이 사용하고자하는 언어별 프레임워크와 도구들을 자유롭게 사용하면 됩니다.

Cloud Client Libraries는 100개 이상의 구글 클라우드 서비스에 사용할 수 있습니다. 가장 잘 사용하는 방법을 더욱 잘 이해할 수 있도록 [홈페이지](https://cloud.google.com/apis/docs/cloud-client-libraries)에 자세한 문서가 있으니 원하는 언어를 선택하여 이를 참고하기만 하면 됩니다.

마지막으로 SDK에는 IDE 창에서 전환할 필요없이 VS Code 및 IntelliJ에서 바로 클라우드 네이티브 앱을 작성, 디버그 및 배포하는 데 사용할 수 있는 확장프로그램인 Cloud Code가 선택적으로 포함되어 있습니다.

이 세가지 외에도 Cloud Shell은 웹브라우저에서 바로 코드를 작성하거나 터미널을 사용할 수 있도록 지원해 줍니다.

이 tool은 구글 클라우드 홈페이지에 접속 후 우측 상단에 ‘cloud shell 활성화하기’ 버튼를 클릭하여 사용 가능합니다.

![gcloud-sdk2](/images/gcloud-sdk2.png)

추가적으로 Google Cloud CLI에서도 별도의 커맨드를 입력하면 cloud shell 사용이 가능한데, 이 기능이 필요하다면 [공식 문서](https://cloud.google.com/shell/docs/using-cloud-shell-with-gcloud-cli)를 참고하면 됩니다.
