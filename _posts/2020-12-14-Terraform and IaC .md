# Terraform and IaC 기초

이번에 테라폼을 이용해서 DB인프라를 구축하는 티켓을 받았다. 살짝 TMI지만 받으면서 포지션도 원래 DevOps에서 약간 Back-end / Plug in 서비스 개발로 변경되었다. 지난번에 DB쪽 모니터링을 담당했는데 그 때 얻은 지식을 바탕으로 구축하는걸 그나마 잘 관찰..할 수 있게 되었다(ㅋㅋㅋㅋ디비 인프라 구축은 내가 아직 역량이 딸려서 못하고, 구축하는걸 잘 관찰하고 서폿 코드 짜는.. 조교 역할 정도..의 티켓). 그래도 팀 리더님이 시간 내주셔서 Terraform 기초 강의까지 해주셔서 안잊어버리고 싶어서 글로 남겨가며 복습 복습.

### Terraform?

기본적인 컨셉은 원하는 인프라를 코드로 서술할 수 있다는 것.

IaC를 가능하게 해주는 도구 중에 하나다.

**IaC는 뭔데? (Infrastructure as a Code)**

인프라 구성을 소프트웨어와 같이 코드를 이용해 처리하는 것을 지칭한다. IaC는 클라우드와 Legacy한 인프라 자원의 어떻게 보면 아주아주 가장 큰 차이를 가져다주는 개념이다. 

Cloud 시대에 도입하면서, 그전과는 달리 '인프라' 를 API로 제어할 수 있는 '인프라 코드 전환' 의 세계가 열렸다. 예를 들면, AWS CLI를 켜서 `나 EC2서버 하나 생성해줄래?`에 해당하는 command를 날리면 그걸 통해서 가상화된 서버가 하나 생성되는 것이다. 이 말을 다르게 해석하면, API를 활용해서 인프라 구성을 코드 형태로 관리하기 시작하게 되었다는 것이다. 이게 IaC고,  Cloud as a Service다. 기존 Legacy방식과의 가장 큰 차이점이기도 하다.

**IaC 도구의 네가지 종류**

IaC Tool은 그 특성에 따라서 4가지 정도로 나뉜다. 

- **Ad-hoc**

    단발성. 즉 script로 구현하는 방식이다. Ad-hoc 자체의 뜻은 "이것을 위해" "특별한 목적을 위해서" 인데 (태어나서 이런 단어 처음봄..) 어쨌든 일회용이다. 

    아무래도 일회용이고 스크립트 구현방식이다 보니 단점이 많은데

    작성한 script를 해당 장비로 가서 직접 수행  / 모든 것을 직접 개발하고 직접 구현해야함 / 복잡한 형상과 설정이 필요하면 구현의 난이도가 올라감 /  코드 유지보수 문제로 귀결됨 / 스크립트를 작성한 엔지니어 말고는 알아볼 수 없는 상황을 초래함 / 서버 들어가서 하나하나 설치함..

    주르륵 나열해서 써놔서 안중요해 보이지만, 코드 유지보수 면에서 인프라 엔지니어들한테 부담이 된다고 하셨다. 팀리더님이 예전에 Ad-hoc으로 관리하시면서 있었던 일화도 말씀해 주셨는데, 한줄로 요약하자면 뭔지도 모르고 그냥 무작정 설치했다는 이야기였다. (하여간 결론적으로 유지보수가 똥망할 수 있다는 이야기)

- **Configuration Management**

    인프라 배포 이후에 앱이 동작하기 위해서 필요한 서버 설정 작업을 어느 정도 자동화한 방식이다.

    Configuration Management를 하기 위한 툴이 있고, 그럼 Ad-hoc방식보다는 조금 더 견고하게 선언을 해서 사용할 수 있다.

    Ad-hoc에 비해 장점이 몇가지가 존재한다.

    1) 중앙에서 대규모 분산 환경 관리가 가능하도록 설계됨.

    (서버에 하나하나 들어가서 설치하는게 아니라 서버-클라 존재)

    2) 멱등성을 보장해준다.

    ```
    **멱등성? (Idempotence)
    한글도 영어도..**태어나서 처음 들어본 말22.. 

    code를 여러 번 수행해도 동일한 결과를 가져올 수 있어야 한다는 개념이다.
    ```

    멱등성 보장을 아주 쉬운 예시로 설명해주셨다.

    a → b로 copy하는 코드를 만들고, 이걸 2번 수행했을 때, 첫번째 copy 후에 두 번째로 copy를 시도했을 때, 이미 값이 있으므로 fail이 떨어지게 해주는게 멱등성 보장이다.

    CM tool의 예시로, 익히 알고 있는 ANSIBLE, chef(기능이 많음), saltstack.. 등이 있다. 

    ```
    여담) Ansible의 이모저모

    Ansible 이 인기를 끌게 된건 '구조의 단순함' 때문이다. 
    SSH를 이용해서 명령 전달(SSH 프로토콜 사용 안하는 것이 추세이거늘..) 
    SSH만 이용해서 명령을 전달하므로, Client agent가 불필요하고, 네트워크만 확보되어 있으면 바로 도입해서 사용이 가능하다~

    단점은 다른 툴에 대비해서 성능이랑 보안에 취약하다. 
    (그래도 인기가 있는건 configuration만 잘 해주면 성능은 좀 괜찮아~ 봐줄께 여서 라고 함)
    ```

- **Server Template**

    Template 방식으로 말아서 배포하는 구조를 가지는 것들을 말한다.

    소프트웨어 수행 시에 필요한 설정, 패키지, OS 이미지를 스냅샷해서 이미지화한 후 배포하는 방식이다. 익히 알고 있는 Docker가 이 템플릿 방식에 속한다. 

    IaC 도구를 통해서 해당 이미지를 배포하고 구성하는 방식이 server template 방식이다. 

- **Server Provisioning**

    여지껏 위 3가지는 서버 하나 내에서의 configuration이었지만, Server Provisioning 방식은 그것보다 한 단계 위의 인프라 환경 자체를 code 형태로 구성하는 방식이다. 서버, Virtual network VPC, Subnet.. 과 같은 인프라 환경 자체를 코드로 관리하도록 하는 방식.  

    오늘 내가 배운 Terraform이랑 익숙한 AWS cloudformation 등이 이 Server Provisioning 방식에 속한다.

### IaC 특징

**반복 가능성**

- 인프라 형상을 정의한 source code만 있으면 언제든지 동일한 인프라를 제작 가능하다. 만약에 동일한 형상의 인프라 환경이 여러 셋이 필요할 경우에, environment 설정을 분리하여 조합해서 인프라를 제작하는 것도 가능하다.

    여러 명의 시스템 엔지니어가 인프라 제작에 뛰어들거나, 중간에 사람이 교체되어도 동일한 인프라를 제작할 수 있다. 

- 형상관리 versioning

     인프라 형상도 코드화 되었기 때문에 git같은 관리 툴을 이용해 versioning해서 관리가 가능하다. (version control system) 원래 시스템 엔지니어는 인프라 히스토리를 알고 있을수록.. 개쩌는 SE가 된다고 한다. 노하우라는게 있기 때문! 근데 버전 컨트롤이 가능하고, 버전 별 인프라 구축 기록이 남으면서 이전 사람이 없어도, 인프라 히스토리를 쉽게 알게 된다. 

**일회성 Disposability** 

- 인프라에 걸리는 의존성을 최소화 시켜줌!! 이건 생성과 수정에 대한 작업 부담이 낮아진다는 이야기다.

    운영팀에 속하는 DevOps도 한 두달 해보고, 지금 완전 개발팀에도 속해보고 있는데 이 두개 입장에서 봤을 때 두쪽 모두에게 인프라 환경을 바꾸는건 큰 부담이다. 그냥 혼자 끄적거리면서 콘솔 들어가서 슝~ 지우고 슝~ 만들면 되는건 실습환경일 때 뿐이다. 회사에서는 권한 하나 수정할래도 티켓 남기고, 담당자 지정하고, 승인받고, 그다음에 인프라를 변경해주신다. DB를 통째로 바꿔야 하거나, VM을 통째로 갈아끼우거나.. 그런 큰 스케일의 인프라 변경은 작업 부담이 클 수밖에 없다.

    개발자가 언제나 환경을 쉽게 생성하고 수정하도록 해주는 특징은 진짜 큰 장점인 것 같다.

---

그래서 테라폼은? 

**Terraform이 동작하는 특징**

- `Code - Plan - Apply` 의 과정을 거친다.
- Multi cloud providers를 서포트한다.
- cloud service 뿐만 아니라 MySQL, Docker 등 다양한 플러그인을 서포트해준다.
- `HCL` 사용

    configuration을 위한 전용 언어이다. 리소스를 블락으로 지정하고, 키=밸류 형태로 리소스 속성을 정의하는 구조!

---

그래서 진짜 초간단 실습을 하나 찾아서 해봤다.

## Terraform을 이용한 AWS EC2배포

### 1) Terraform 설치 EC2용 SSH key pair정의

테라폼을 한번 써보려고  [https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code) 를 참고해서 AWS에 서버를 배포해봤다.

```
mzc01-jiyoon@MZC01-JIYOON ~ % brew install terraform // mac OS 이용자는 brew로 간단하게 설치 가능
Updating Homebrew...
==> Downloading https://homebrew.bintray.com/bottles-portable-ruby/portable-ruby-2.6.3_2.yosemite.bottle.tar.gz
##################################################################################################################################################################################################################################### 100.0%
==> Pouring portable-ruby-2.6.3_2.yosemite.bottle.tar.gz
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> New Formulae
attr              aws-console       docui             duf               inframap          json5             kubevela          lua@5.3           magic_enum        oakc              terracognita      terraform@0.13    vc
==> Updated Formulae
Updated 1010 formulae.
==> Deleted Formulae
dtrx                                                                                                                   unrar

==> Downloading https://homebrew.bintray.com/bottles/terraform-0.14.2.catalina.bottle.tar.gz
==> Downloading from https://d29vzk4ow07wi7.cloudfront.net/652bd739beb1f41d55f44d35bdce12c1b71a566efad21375079fc35bb89e34e5?response-content-disposition=attachment%3Bfilename%3D%22terraform-0.14.2.catalina.bottle.tar.gz%22&Policy=eyJTdG
######################################################################## 100.0%
==> Pouring terraform-0.14.2.catalina.bottle.tar.gz
🍺  /usr/local/Cellar/terraform/0.14.2: 6 files, 63.9MB

mzc01-jiyoon@MZC01-JIYOON ~ % terraform version
Terraform v0.14.2
```

### 테라폼에 등장하는 몇 가지 등장인물들

**Provisioning**

어떤 프로세스나 서비스를 실행하기 위한 준비 단계

네트워크, 컴퓨팅 자원을 준비하는 작업이랑 준비된 컴퓨팅 자원에 site package, application 의존성 준비하는 단계로 나뉘어진다.

**Provider**

테라폼이랑 외부 서비스를 연결해주는 기능을 하는 모듈이다.

ex) AWS provider를 설정해서 AWS 서비스의 컴퓨팅 자원을 생성함.

위에 언급했지만 AWS , GCP 같은 클라우드 provider들도 있고, Github, DataDog같은 서비스, MySQL, docker(오..) 도 있음.

```
provider "aws" {
	access_key = "xxxxxx"
	secret_key = "xxxxxx"
	region = "ap-northeast-2"
}
```

전체 목록 링크 : [https://www.terraform.io/docs/providers/index.html](https://www.terraform.io/docs/providers/index.html)

**terraform 프로젝트 init**

```
mzc01-jiyoon@MZC01-JIYOON web_infra % terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v3.21.0...
- Installed hashicorp/aws v3.21.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

**Resource**

EC2 서버에 SSH로 접속을 해야하기 때문에, key pair generation이 필요하다. 기존에 만들어둔 키가 있으면 사용해도 되고, 나는 없어서 ssh-keygen 명령어로 만들어줬다. 이 키는 `/$HOME/.ssh/` 경로에 public key와 private key로 저장된다. 

```
mzc01-jiyoon@MZC01-JIYOON web_infra % ssh-keygen -t rsa -b 4096 -C "email@email.com" -f "$HOME/.ssh/seb-admin" -N ""
Generating public/private rsa key pair.
Your identification has been saved in /Users/mzc01-jiyoon/.ssh/seb-admin.
Your public key has been saved in /Users/mzc01-jiyoon/.ssh/seb-admin.pub.
The key fingerprint is:
SHA256:xxxx email@email.com
The key's randomart image is:
```

```
mzc01-jiyoon@MZC01-JIYOON web_infra % cat web_infra.tf
resource "aws_key_pair" "web_admin" {
	key_name = "web_admin"
	public_key = file("/Users/mzc01-jiyoon/.ssh/seb-admin.pub")
}
```

**Plan**

```
mzc01-jiyoon@MZC01-JIYOON web_infra % terraform plan

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_key_pair.web_admin will be created
  + resource "aws_key_pair" "web_admin" { // tf file에다가 선언해놓은 상태랑 같게 만들께~라는 뜻
      + arn         = (known after apply)
      + fingerprint = (known after apply)
      + id          = (known after apply)
      + key_name    = "web_admin"
      + key_pair_id = (known after apply)
      + public_key  = "ssh-rsa == xxx...email@email.com"
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

테라폼의 동작 방식은 "리소서를 선언적으로 기술하기" 다. 

.tf파일에 기술되어 있는 모든 리소스를 읽어들이고, 이 리소스들이 먼저 존재하는 상태를 가정한다. (ideal한 상태) 아직 이 리소스가 실제로 생성된 적은 없으므로, provider에 지정한 AWS계정에는 존재하지 않는다. 

테라폼은 이 선언해놓은 ideal한 상태를 실제 상태랑 동일하게 만드는게 목표다. (약간 K8S랑 컨셉이 비슷해보임.. K8S도 yaml파일로 선언해놓고 쿠버가 비교를 통해 그 상태에 도달시켜줌)

**Apply**

```
mzc01-jiyoon@MZC01-JIYOON web_infra % terraform apply

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_key_pair.web_admin: Creating...
aws_key_pair.web_admin: Creation complete after 0s [id=web_admin]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
mzc01-jiyoon@MZC01-JIYOON web_infra %
```

### 2) SSH 접속 허용을 위한 Security Group만들기

**Code** tf파일에 계속 계속.. 추가 

```
mzc01-jiyoon@MZC01-JIYOON web_infra % vim web_infra.tf

//아래쪽에 내용 추가
resource "aws_security_group" "ssh" {
	name = "allow_ssh_from_all"
	description = "Allow SSH port from all"
	ingress {
		from_port = 22
		to_port = 22
		protocol = "tcp"
		cidr_blocks = ["0.0.0.0/0"]
	}
}

// 이미 클라우드 기본 VPC에 기본으로 지정되어 있는 security group을 불러옴. 이 리소스를 데이터 소스로 불러오는 기능을 제공한다. 
data "aws_security_group" "default"{
	name = "default"
}
```

**Plan**

```
mzc01-jiyoon@MZC01-JIYOON web_infra % terraform plan
aws_key_pair.web_admin: Refreshing state... [id=web_admin]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_security_group.ssh will be created
  + resource "aws_security_group" "ssh" {
      + arn                    = (known after apply)
      + description            = "Allow SSH port from all"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0", //모든 ip 허용
                ]
              + description      = ""
              + from_port        = 22
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 22
            },
        ]
      + name                   = "allow_ssh_from_all"
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + vpc_id                 = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

어떤 리소스를 정의할 때 다른 리소스들의 속성을 참조할 수 있다. 그냥 블라블라~!@@#! 나열할 수도 있지만 속성으로 참조하는게 더 좋은 이유는 두 리소스 간에 의존 관계가 생기기 때문이다. 

처음에 tf 파일을 작성하면서, 그럼 security group을 instance보다 위에 선언해야 하나? 라는 의문점이 들었는데, 테라폼은 소스코드의 순서가 아니고 그래프 모델로 이런 의존 관계를 정의해서 생성할 순서를 결정한다고 한다.

내가 따라해본 예제에서는 aws_key_pair.web_admin 리소스가 더 먼저 생성해주는걸 보장해줌. 

**Apply**

마지막으로 apply해주면 AWS console에 들어가서 이리보고 저리 봐도 내가 만든 all ip 허용 + 기본 security group + t2.micro에 내가 지정한 ami 조합의 인스턴스가 딱 나타난걸 볼 수 있다.

```
mzc01-jiyoon@MZC01-JIYOON web_infra % terraform apply
```

![aws_instance](https://user-images.githubusercontent.com/25656426/102163175-f97f0100-3ecd-11eb-998a-b4a7d1364d5f.png)


### 테라폼을 사용하면서 주의해야 할 점

- Sync 맞추기

테라폼으로 인프라를 관리한다고 정하면, 이후에 manual 하게 콘솔에 들어가서 수정하거나 한  부분은 테라폼에 반영이 안된다. 테라폼으로만 쓰던, 콘솔로만 쓰던 하나만 정하던가 / manual하게 수정한 부분을 테라폼 코드에 반영하던가 해서 싱크를 맞춰야한다. 

- 하나의 tf파일에 모든 인프라 구조를 다 선언하지 않기

모듈 단위로 안나누고 내가 한 실습처럼 하나 tf파일에 인프라를 다 선언하면 유지보수가 어려운 난관에 봉착할수밖에 없다. 코드 짠사람이야 금방 수정하겠지만 아니면 그 모든 코드를 다읽어야 하니깐 ... 우엑 모듈을 잘게잘게 나눠서 설계를 먼저 잘 하는게 중요한 듯 싶다. 

### 마치며

우와 테라폼은 진짜 신기.. 하다. 기본 vim 가지고 몇 자 끄적였더니 AWS 콘솔에 인스턴스가 나타났다. 신세계 그 자체다.

HCL을 난 어제 처음봤지만, 테라폼 "코드를 작성하는 것 자체" 는 진입장벽이 높아보이진 않는다. 설계가 중요해보임. 새로운걸 알게 되어서 또 겁나 뿌듯함을 느낀다. 그전에는 뭘 해봐야하는지 공부해봐야하는지 아예 소스조차 없었는데 가닥이 잡히는 기분. 

요즘 클라우드 관련주를 모으고 있는데 하나 배우고 주식 사고 또 하나 공부하고 뽀개면 주식 사고하는게 재밌다.  고딩때는 스터디플래너에 완료 줄 하나 긋던걸 이제 어른이 되었으니 공부 시작하면 주식을 산다. 기분으로 주식 매수하는 사람 나야나.. 근데 테라폼 hashicorp 주식은 왜 취급 안하는걸까 진짜 살텐데..

하여간 더 advanced 하게 테라폼을 이용해서 구축하게 되면 또 포스팅 해야겠다.

### 출처

- 쓰면서 헷갈리는게 있어서 구글 검색했는데 IaC 4가지로 분류한 방식이 안나오는걸 보니깐 가르쳐주신 분이 생각하셨나??!!보다. 그렇다면 모든 저작권은 JH song님께..
- 실습은 [https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code) 여기 링크 참고했다.