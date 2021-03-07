---
date: 2021-01-19
title: "Azure Load Balancer 개념과 AWS ELB와의 비교"
categories: 
  - Azure
tags:
  - Network
  - Load Balancer
  
thumbnail: https://user-images.githubusercontent.com/25656426/110240762-c27e3000-7f90-11eb-9604-6f7bf5d79401.png

---

# Azure Load Balancer 개념과 AWS ELB와의 비교

## LoadBalancer (LB)

Azure Service 탐구 제4탄. Load Balancer. 이번 포스팅에서 이야기할 내용은 다음과 같다.

- Azure의 Load Balancer란
- Azure Load Balancer의 유형
- Azure 의 Application Gateway
- AWS ELB와의 비교

### Azure의 **Load Balancer란?**

Azure Load Balancer는 OSI 모델 layer4 에서 작동하며, Backend Resource 나 server group에서 들어오는 네트워크 트래픽 부하를 효율적으로 분산시키는 장치이다.

**Who’s balanced?**

LB는 load balancer의 front end로 들어오는 inbound flow를 backend instances로 pool해줌. backend instance는 Azure vm이나, vm Scaleset의 인스턴스들이 될 수 있다.

**How balanced?**

1.  **Inbound dedicated load balancers (Private LB)**

    지정한  ①`**load balancing rules**`,  ② `**health probes**`에 따라서 부하가 분산된다.

    ---

    ①`**load balancing rules**`

    - backend pool 내의 모든 인스턴스에 들어오는 트래픽이 배포되는 방법을 정의하는데 사용된다.
    - 지정된 frontend ip구성, 포트를 backend ip주소와 포트에 매핑한다.

    ![Untitled](https://user-images.githubusercontent.com/25656426/110240782-daee4a80-7f90-11eb-8649-0f1e61ce7d85.png)

    - LB rules를 지정하기 위해서, Azure portal 에서 몇 가지 지정해야 할 요소는 다음과 같다.
        - **`Backend port`**

            Backend pool의 어떤 포트로 트래픽 분산할지 정의

        - **`Backend pool`**

            분산될 트래픽 대상(ip configuration은 같은 vn내 위치해야함.

            `vnet(1):Backend Pool(N)`관계이며, pool 내의 vm들은 같은 region 내에 위치해야함.

        - **`Health probe`**

            Backend pool의 상태 검사

        - `**Session persistance**`

            트래픽이 세션 기간 동안 Backend pool에 있는 가상머신 중 같은 머신에서 처리되어야 하는지 지정한다 3가지의 옵션이 있다.

            - `None` : 요청 모든 가상머신에서 처리
            - `Client IP` : 동일한 client ip 주소의 연속적인 요청이 동일한 가상머신에서 처리되도록
            - `Client IP & Protocol`

                동일한 client ip 에다가 더해서 protocol 요청까지 동일한 가상머신에서 처리되도록 결정

        - `**Idle timeout(minutes)**`

            keep-alive messages를 보내기 위해 클라이언트에 의존하지 않고 TCP HTTP connection을 얼마나 열어둘지를 결정하는 옵션. 

        - **`TCP reset`**
        - **`Floating IP`**

            동적 ip 옵션을 지정하지 않으면  azure는 자체 traditional mapping scheme 에 따라서 load balancing ip address를 노출함. 옵션을 활성화하면 프론트엔드 ip랑 동기화함.

        - `**Outbound source network address translation**`

            LB의 frontend 에 지정된 public ip를 사용하도록 backend pool instance에 대해 아웃바운드 규칙 구성

        ---

    ② `**Health Probes**`

    - Health Probes는 부하를 분산하기 전, backend pool의 인스턴스들이 healthy 한지 검사하기 위해 사용된다.
    - `HTTP/HTTPS` 로 health check를 날리는 경우, probe endpoint에서 `200` 이외의 응답코드를 반환하면 가동 중단 상태로 표시한다.

    Azure portal 에서 5가지 요소를 설정해야한다. 

    - `**Health probe name**`
    - **`Protocol`**

        어떤 프로토콜 사용해 health check할 것인가 (TCP, HTTP, HTTPS) 를 선택하는 옵션

    - **`Port`**

        health signal의 목적 포트 (Application 포트와 같을 필요는 없다!!)

    - **`Interval`** (단위 : seconds)

        몇초 단위로 health check 날릴껀지

    - `**Unhealthy threshold**`

        최대 연속 probe failure 수 (이거 넘으면 인스턴스 비정상으로 간주)

    Basic SKU(Pricing tier) 의 경우 HTTPS health check protocol을 지원하지 않는다.

    ---

2. **Outbound dedicated load balancers (Public LB)**

    ![Untitled 1](https://user-images.githubusercontent.com/25656426/110240745-b72b0480-7f90-11eb-8507-0e81d67838a7.png)

    Outbound load balancing 은 어떤 vm이 어떤 public ip address로 할당될지, 아웃바운드 connection idle timeout 등을 가능하게 해준다.

### 유형

Azure Load Balancer를 목적 별로 분류한 유형은 2가지. 아래와 같다.

![Untitled 2](https://user-images.githubusercontent.com/25656426/110240748-b98d5e80-7f90-11eb-9693-05828207354a.png)

## Azure Application Gateway

**Azure Application Gateway**

위의 Load Balancer와 기능적으로 동작하는건 같으나, OSI 7계층에서 작동하는 LB이다. 

웹 어플리케이션 수준에서 트래픽을 관리할 수 있도록 하는 트래픽 부하 분산 장치이다.

**Who’s balanced?**

AG는 load balancer의 front end로 들어오는 inbound flow 중에서도 **`URI 경로`**, `**host header**`와 같은 HTTP 요청의 추가적인 특성을 기반으로 라우팅해서 backend instances로 pool 해준다. Azure LB와 마찬가지로, Backend instance는 Azure vm이나, VM Scaleset의 인스턴스들이 될 수 있다. 

![Untitled 3](https://user-images.githubusercontent.com/25656426/110240754-bbefb880-7f90-11eb-842c-cbfd427255b4.png)

### AWS ELB와의 비교

AWS에서의 LB hands on 경험이 없었어서, 이번 기회로 콘솔 살펴보면서 AWS vs Azure LB 비교를 한번 해봤다. 

![Untitled 4](https://user-images.githubusercontent.com/25656426/110240762-c27e3000-7f90-11eb-9604-6f7bf5d79401.png)

이렇게 Azure LB 정복 끝~