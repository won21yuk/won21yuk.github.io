---
title: Terraform practice - 설치 및 테스트
categories: [Infra, Terrform]
---

테라폼(Terraform)은 IaC에서 가장 많이 사용되는 툴이다. 사실 이론적인 내용을 먼저 공부하려고 했는데, 이론적인 공부를 하는것보다 일단 사용해보고 하는게 좋을거같다는 충동적인 생각이 들었다. 그래서 우선 공식 홈페이지를 보면서 설치 작업을 수행하고, 이후 실습을 적당히 진행해본 후 테라폼에 대한 설명이 담긴 포스팅을 별도로 작성하려 한다. 이번 포스팅에서는 설치와 간단한 테스트 작업만 수행할 것이다.

# 설치

테라폼 홈페이지에 가면 맥 OS, 윈도우 OS, LINUX에서 설치하는 방법이 상세하게 나와있다. 난 윈도우를 사용하기 때문에 공식홈페이지 안내에 따라 Chocolatey 패키지 매니저 설치를 우선 진행한다.

```text
# in cmd

# 관리자의 권한으로 CMD창을 키고 입력
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

# Chocolatey 버전 확인
choco -V
# 1.2.0
```

Chocolatey를 설치하고 나면 이를 통해 테라폼을 설치할 수 있다.

```text
choco install terraform
```

테라폼에서 사용가능한 커맨드는 `terraform -help` 명령어로 확인할 수 있고 그 내용은 아래와 같다.

![terraform-practice-tutorial0](/images/terraform-practice-tutorial0.png)

# GCP와 연결

![terraform-practice-tutorial1](/images/terraform-practice-tutorial1.png)

GCP와 테라폼을 연동하기 위해서는 GCP Project가 하나 있어야하고, GCE API가 활성화 되어 있어야하며, GCP 서비스 계정 키가 필요하다. 앞의 두개는 GCP를 사용해봤다면 익숙할 것이기 때문에 넘어가고 서비스 계정 키를 만드는 것만 해보겠다.

콘솔에 있는 검색창에 service account를 치면 서비스 계정 페이지로 바로 이동이 가능하다.

![terraform-practice-tutorial2](/images/terraform-practice-tutorial2.png)

여기서 서비스 계정 만들기를 클릭하면 서비스 계정을 생성할 수 있는 페이지로 이동한다.

![terraform-practice-tutorial3](/images/terraform-practice-tutorial3.png)

원하는 서비스 계정의 이름을 설정해주고 ‘만들고 계속하기’ 버튼을 클릭한다.

![terraform-practice-tutorial4](/images/terraform-practice-tutorial4.png)

역할을 ‘편집자’로 변경해준 후, 완료버튼을 클릭한다.

![terraform-practice-tutorial5](/images/terraform-practice-tutorial5.png)

그러면 메인페이지에 아래와 같이 서비스계정이 생성된것을 확인할 수 있다. 여기서 맨 우측의 점표시를 클릭하고 키관리로 들어간다.

![terraform-practice-tutorial6](/images/terraform-practice-tutorial6.png)

그 다음 키추가를 누르고 새 키 만들기를 클릭한다. 그리고 Json 유형으로 선택한 후 만들기를 클릭한다.

![terraform-practice-tutorial7](/images/terraform-practice-tutorial7.png)

그러면 비공개 키가 컴퓨터에 저장됐다는 메세지가 뜨고 이 창을 닫으면 json 파일이 자연스럽게 로컬 pc에 다운 받아진 것을 확인 할 수 있다.

![terraform-practice-tutorial8](/images/terraform-practice-tutorial8.png)

이 서비스 어카운트 키는 수많은 리소스를 생성할 수 있는 권한과 관련된 것이기 때문에 해당 키가 노출됐을 때 발생할 수 있는 리스크가 너무 크다. 따라서 보안에 유의할 필요가 있다.

마지막으로 테라폼은 자체 작업 디렉토리를 기준으로 작동하기 때문에 관련 디렉토리를 생성하고 tf파일을 생성해줘야한다.

```text
mkdir learn-terraform-gcp
cd learn-terraform-gcp
copy con main.tf
# copy con 명령어를 치고 한번더 엔터를 쳐줘야 생성이 완료된다.
```

이 작업까지 완료되면 홈페이지에 나와있는 간단한 예제로 tf파일을 작성해 볼 것이다. 이때 해줘야하는 작업은 자신의 프로젝트 아이디와 서비스 계정 키 파일을 입력해주는 것이다.

```text
# main.tf

terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.5.0"
    }
  }
}

provider "google" {
	# 앞서 받은 서비스 계정키 퍼알(*.json)의 이름을 입력
  credentials = file("terraform-practice-369907-098e7b100728.json")

	# 자신의 프로젝트 아이디 입력
  project = "terraform-practice-369907"
  region  = "asia-northeast3"
  zone    = "asia-northeast3-c"
}

resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
```

# 테스트

홈페이지에 나와있는 샘플 코드들을 활용해 정상적으로 테라폼이 작동하는지 테스트를 진행해본다.

## 디렉토리 초기화

새로운 설정 파일을 만들거나 기존 설정들을 체크아웃하려는 경우에는 항상 작업 폴더를 초기화하기 위해 `terraform init`명령어를 입력해야한다. 이 단계에서는 설정에 정의 된 프로바이더를 다운로드 한다.

![terraform-practice-tutorial9](/images/terraform-practice-tutorial9.png)

테라폼은 Google 프로바이더를 다운로드하여 현재 작업 디렉토리의 숨겨진 하위 디렉토리인 .terraform에 설치한다. terraform init 명령은 설치된 프로바이더 버전 테라폼을 인쇄한다. 또한 테라폼은 .terraform.lock.hcl이라는 이름의 잠금 파일을 생성하는데, 이 파일은 모든 테라폼 실행이 일관되도록 하는 데 사용되는 정확한 프로바이더 버전을 지정한다. 그리고 설정에 사용되는 프로바이더를 업그레이드할 시기를 제어할 수도 있다.

![terraform-practice-tutorial10](/images/terraform-practice-tutorial10.png)

## 구성 형식 지정 및 유효성 검사

기본적으로 모든 구성 파일에 대해 일관된 형식을 사용하는 것이 좋다. `terraform fmt` 명령은 가독성과 일관성을 위해 현재 디렉토리 내의 구성을 자동으로 업데이트한다. 결과적으로는 수정한 파일의 이름이 출력되며, 이 경우 main.tf파일을 수정했기 때문에 해당 파일이 출력되었다.

![terraform-practice-tutorial11](/images/terraform-practice-tutorial11.png)

또한 `terraform validate` 명령을 사용하여 tf 파일의 구성이 구문적으로 유효하고 내부적으로 일관적인지를 확인할 수 있다. 이 경우에는 main.tf 파일 구성이 유효하므로 Success 메세지를 반환한다.

![terraform-practice-tutorial12](/images/terraform-practice-tutorial12.png)

## 인프라 생성

`terraform apply` 명령을 통해 현재 구성을 적용한다.

![terraform-practice-tutorial13](/images/terraform-practice-tutorial13.png)

출력되는 내용을 좀 더 자세히 살펴보면, 테라폼이 인프라를 만들기위해 수행할 작업들을 설명하는 일종의 실행 계획이 표시되며 출력 형식은 git과 같은 도구에서 생성되는 diff 형식과 유사하다. resource “google_compute_network” “vpc_network” 옆에 +가 있는데 이는 테라폼이 이 리소스를 생성함을 의미한다. 그 아래에는 설정될 속성값이 표시된다. 만약 속성 값이 known after apply라고 나온다면, 리소스가 생성될 때까지 값을 알 수 없다는 것을 의미한다.

그리고 마지막에 보면 이 액션을 수행할 것인지에 대해 승인을 기다린다. 만약 실행계획이 원하는 형태가 아니라면 여기서 중단하면 인프라를 변경하지 않고 멈출수 있다. 이 계획대로 실행하기를 원한다면 프롬프트에 yes를 입력하면 인프라 생성작업을 본격적으로 수행한다. 테라폼이 VPC 네트워크를 프로비저닝하는 데는 몇분 정도 소요된다.

![terraform-practice-tutorial14](/images/terraform-practice-tutorial14.png)

Apply complete가 뜨면 테라폼을 사용하여 인프라를 만드는데 성공한 것이다. 이제 GCP 콘솔에 들어가서 프로비저닝한 네트워크를 확인해본다.

![terraform-practice-tutorial15](/images/terraform-practice-tutorial15.png)

terraform-network라는 이름으로 VPC 네트워크가 추가된 것을 확인할 수 있다.

## 상태검사

구성을 적용하면 테라폼은 terraform.tfstate라는 파일에 데이터를 기록한다.

![terraform-practice-tutorial16](/images/terraform-practice-tutorial16.png)

테라폼은 관리하는 리소스의 ID와 속성을 이 파일에 저장하여 해당 리소스를 업데이트하거나 삭제할 수 있다. 테라폼 State file은 테라폼이 관리하는 리소스를 추적할 수 있는 유일한 방법이며, 중요한 정보를 포함하는 경우가 많으므로 상태 파일을 안전하게 저장하고 인프라를 관리해야하는 신뢰할 수 있는 팀원에게만 배포해야한다.

`terraform show` 명령을 사용하면 현재 상태를 확인할 수 있다.

![terraform-practice-tutorial17](/images/terraform-practice-tutorial17.png)

테라폼이 이 네트워크를 생성할 때, 구글 프로바이더로 부터 메타데이터를 수집하고 state file에 이 내용을 기록한다. 나중에 이러한 값들을 참조하여 다른 리소스 또는 아웃풋을 구성하도록 구성을 수정한다.
