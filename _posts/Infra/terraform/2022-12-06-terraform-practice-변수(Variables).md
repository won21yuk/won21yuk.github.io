---
title: Terraform practice - 변수(Variables) 사용
categories: [Infra, Terraform]
---

이전 포스팅에서는 하드코딩된 값을 사용했지만 테라폼은 변수를 사용하여 보다 구성을 다이나믹하고 유연하게 만들 수 있다.

# 입력(Input) 변수 정의

테라폼은 작업하고 있는 디렉토리에 있는 모든 tf파일을 로드하기 때문에 원하는 대로 구성파일의 이름을 정해줄 수 있다.  따라서 learn-terraform-gcp 디렉토리에 변수에 대해 정의할 variables.tf 파일을 생성하고 아래의 내용을 작성한다.

```python
# variables.tf

variable "project" { }

variable "credentials_file" { }

variable "region" {
  default = "us-central1"
}

variable "zone" {
  default = "us-central1-c"
}
```

위의 내용은 Terraform 구성 내에 네 가지의 변수를 정의한다. project와 credentials_file 변수에 빈 블록 {}이 있고 region과 zone 변수에는 기본값(default)을 설정한다. 기본값이 설정된 경우에 변수는 선택 사항이고 그렇지 않으면 변수가 필요하다. 지금 terraform plan을 입력하면, 테라폼은 project 및 credentials_file에 대한 값을 입력하라는 메시지를 표시한다.

![terraform-practice-variables0](/images/terraform-practice-variables0.png)

## 구성(configuration)에 변수 사용하기

main.tf에 새로운 변수를 사용하여 GCP 프로바이더 구성을 업데이트해준다.

```python
# main.tf

terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}

provider "google" {
  version = "3.5.0"

- credentials = file("<NAME>.json")
+ credentials = file(var.credentials_file)

- project = "<PROJECT_ID>"
- region  = "us-central1"
- zone    = "us-central1-c"
+ project = var.project
+ region  = var.region
+ zone    = var.zone
}
```

tf 파일에서 변수는 접두사 `var.`로 참조됩니다.

## 변수에 값 할당하기

테라폼은 작업을 실행할 때 terraform.tfvars 또는 *.auto.tfvars와 일치하는 파일을 찾아 자동으로 로드한다. 이를 파일의 값을 변수의 값으로 사용할 수 잇다.

terraform.tfvars 파일을 만들어 아래의 값을 기입한다.

```python
# terrafor.tfvars

project = "terraform-practice-369907"
credentials_file = "terraform-practice-369907-098e7b100728.json"
```

한가지 주의해야할 건 terraform.tfvars 파일에는 credentail 값과 같은 민감한 내용이 담길 수도 있기 때문에 기본적으로 커밋하지 않는 것이 좋다.

이제 terraform apply를 실행한다. 다만 기존 값들을 변수로 바꿔주기만 했을 뿐 실제 구성에 변화가 없기 때문에 아무런 변화가 일어나지 않는다.

![terraform-practice-variables1](/images/terraform-practice-variables1.png)

# 출력(Output) 변수 정의

앞선 단락에서는 입력변수를 정의하고 이를 활용해 테라폼 구성을 매개변수화 했다. 이번 단락에서는 출력 변수를 통해 테라폼 사용자에게 쉽게 쿼리하고 표시할 데이터를 구성한다.

복잡한 인프라를 구축할 때는 테라폼은 수백 수천개의 속성값을 저장한다. 일부 사용자는 그중 몇가지에만 관심이 있을수 있다. output은 표시할 데이터를 지정하는데 이 데이터는 terraform apply 명령이 호출될 때 출력되며 `terraform output` 명령을 사용하여 쿼리할 수 있다.

테라폼이 프로비저닝하는 인스턴스의 IP 주소에 대한 출력을 정의하기 위해 outputs.tf 파일을 생성하여 아래와 같은 내용을 기입한다,

```python
output "ip" {
  value = google_compute_instance.vm_instance.network_interface.0.network_ip
}
```

위의 코드는 “ip” 라는 출력 변수를 정의한 내용이다. 변수 이름이 다른 모듈에 대한 입력(input)으로 사용되려면 테라폼 변수 명명 규칙을 준수해야 한다. value 필드는 compute_instance의 첫 번째 네트워크 인터스페이스 특성의 network_ip값을 지정하고 있다. 여러 출력 변수를 지정하고 싶다면 여러 출력 블록을 정의하면 된다.

출력 값을 사용하기전에 이 구성을 apply해야 한다.

![terraform-practice-variables2](/images/terraform-practice-variables2.png)

이제 `terraform output` 명령어를 가지고 outputs을 쿼리해보면 아래와 같이 ip값만 출력이 된다.

![terraform-practice-variables3](/images/terraform-practice-variables3.png)

# 인프라 삭제

지금까지의 테라폼 튜토리얼동안 만든 리소스를 정리하려면 `terraform destroy` 명령어를 실행하면 된다.

![terraform-practice-variables4](/images/terraform-practice-variables4.png)
