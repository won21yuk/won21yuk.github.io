# [week1] Terraform practice - 구성 변경

작성일시: 2022년 12월 1일 오후 12:09
최종 편집일시: 2022년 12월 1일 오후 2:09

이번 포스팅에서는 인프라 구성을 변경할 것이고, 테라폼 프로젝트에 이 변화를 어떻게 적용하는지를 학습할 것이다.

# 새로운 리소스 생성

이전 포스팅에서는 main.tf에  VPC 네트워크를 생성하는 코드만 작성했었다. 이번엔 인스턴스를 만들어 주기 위한 코드를 main.tf에 추가로 작성한다.

```python
# main.tf

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "3.5.0"
    }
  }
}

provider "google" {
  # GCP에서 받은 서비스 계정키 파일(*.json)의 이름을 입력
  credentials = file("terraform-practice-369907-098e7b100728.json")

  # 자신의 프로젝트 아이디 입력
  project = "terraform-practice-369907"
  region  = "asia-northeast3"
  zone    = "asia-northeast3-c"
}

# VPC 생성
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}

# 인스턴스 생성
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}
```

새로운 인스턴스를 만들기 위해 추가한 리소스에는 name, machine_type, boot_disk, network_interface라는 총 4개의 인수가 포함되어 있다. 리소스를 지원하는 인수들에 대한 정보는 테라폼 공식 홈페이지에 있는 [GCP Provider documentation](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance)에서 확인할 수 있다.

인스턴스는 Debian OS를 사용하고 이전에 생성한 VPC 네트워크를 사용한다. 여기서 주목할 건 `google_compute_network.vpc_network.name`의 형태로 네트워크의 이름 속성을 참조하는 방식이다. 이는 두 리소스 간의 종속성(Dependency)를 설정한 것으로 테라폼은 이를 토대로 적절한 순서로 리소스들이 생성되도록 관리하는 종속성 그래프를 작성하게 된다.

추가로 `access_config` 블록은 인수 없이도 VM 인스턴스에 외부 IP 주소를 제공하여 인터넷을 통해 접근 가능하도록 한다.

한가지 주의할 건 방화벽 규칙을 만들어 허용하지 않는다면, 이렇게 만들어진 인스턴스에 대한 모든 트래픽은 방화벽에 의해 차단된다. 심지어 다른 인스턴스로부터의 트래픽도 차단된다. 트래픽이 해당 인스턴스에 엑세스 가능하게 하려면 `google_compute_firewall` 리소스를 추가해야한다.

이제 `terraform apply` 명령어를 통해 인스턴스를 생성한다.

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled.png)

yes를 입력해주면 Apply complete 메세지가 뜨고, GCP에는 인스턴스가 생성된 것을 확인할 수 있다.

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%201.png)

# 구성 수정하기

테라폼은 리소스를 생성하는 것 외에도 리소스를 변경할수도 있다. 이를 리소스 변경을 테스트 해보기 위해서 앞서 작성한 main.tf 파일에서 vm_instance 리소스 블록에 tags 인수를 추가해보겠다.

```python
# 인스턴스 생성
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
	tags         = ["web", "dev"]
  ...
}
```

이후 `terraform apply` 명령어로 구성 변경을 적용시킨다.

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%202.png)

접두사 `~`는 테라폼이 내부 리소스를 업데이트 한다는 것을 의미한다. yes를 입력해 변경사항을 적용시키고 GCP에서 인스턴스를 확인하면 dev, web 태그가 추가 된 것을 확인할 수 있다.

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%203.png)

# ****destructive change****

destructive change는 프로바이더가 기존 리소스를 업데이트 하지않고 교체하는 변경을 의미한다. 이는 보통 클라우드 프로바이더가 구성에 명시된 방식으로 리소스를 업데이트하는 것을 지원하지 않기 때문에 발생한다.

가장 대표적인 것이 인스턴스의 디스크 이미지를 변경하는 작업이다. 이를 테스트 해보기위해 main.tf 파일에서 boot_disk 블록을 편집해 보겠다.

```python
boot_disk {
     initialize_params {
			# Before
      # image = "debian-cloud/debian-11"
			# After
        image = "cos-cloud/cos-stable"
     }
   }
```

위의 코드는 부팅 디스크를 Debian 11에서 구글의 컨테이너 최적화 OS로 변경한 것이다. 이제 terraform apply 명령어로 구성 변경을 적용한다.

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%204.png)

접두사 `-, +`는 테라폼이 리소스를 업데이트하지 않고 리소스를 삭제한 후 다시 작성함을 의미한다. 테라폼과 GCP 프로바이더가 이러한 세부 사항을 처리하고 실행 계획은 테라폼이 수행할 작업을 보고한다.

이 실행 계획은 디스크 이미지 변경이 인스턴스를 강제로 교체한 수정 사항임을 보여준다. 이를 빨강색 텍스트로 표현하고 있다. yes를 입력하여 구성 변경을 적용시켜주면 인스턴스 삭제작업이 일어나고 새로이 인스턴스가 생성되는 작업이 수행됨을 확인할 수 있다. 

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%205.png)

이를 GCP에서도 확인할 수 있다. GCP에서는 인스턴스가 재생성되면 내부 ip가 변하는데 테라폼을 실행 후 10.178.0.2에서 10.178.0.3으로 변한 것을 알 수 있다.

- Before

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%206.png)

- After

![Untitled](%5Bweek1%5D%20Terraform%20practice%20-%20%E1%84%80%E1%85%AE%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20bbb286f3c14648bd9b5d3ed955ccfd4f/Untitled%207.png)