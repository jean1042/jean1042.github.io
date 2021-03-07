---
date: 2020-11-19
title: "Kubernetes Service Account를 이용한 Authentication (RBAC)"
categories: 
  - Kubernetes
tags:
  - RBAC
  - Kubernetes
  - Authentication
  
thumbnail: https://user-images.githubusercontent.com/25656426/110238697-67474000-7f86-11eb-9f1b-e7a330e50876.png

---

# 쿠버네티스 Service Account를 이용한 Authentication (RBAC)

인턴 생활을 진짜 온 머리를 다 써서 열심히 하고 있는 회사에서는 프로덕트를 Kubernetes (이하 K8S) 기반으로 운영/배포한다. 이중에서 Kubernetes cluster 내 namespace에 access 할 수 있는 유저를 다르게 설정해 특정 namespace를 넘어선 pod나 타 object들에는 접근하지 못하도록 하는 이슈의 사전 리서치를 맡았다.   

K8S는 모든권한을 role base로 할당하기 때문에 K8S 클러스터 내의 user개념과 실제 actual human의 사용자 개념은 다른데, 그걸 모르고서 `"아니 user한테 권한 할당 분명 해줬는데 왜 권한이 없다는거야?"` 는 잘못된 무한루프 때문에 시간을 오래 잡아먹게 되었다.

나처럼 오랫동안 고민하는 사람이 없길 바라며 이번 글에서 K8S authentication에는 누가 관여해서, 어떤 메커니즘으로 이루어지고 있는건지 써보려고 한다. 

우리는 클러스터 내에 kubectl을 구축하고 `kubectl get pods -n xxx` 와 같은 커맨드를 날린다. 여기서 이 커맨드는 대체 누구한테 요청하는거지? 라는 궁금증이 생긴다. 답은 kubernetes API 서버에 날리는게 맞다. kubectl client와 API 서버는 최종 사용자, 클러스터의 다른 부분 그리고 외부 컴포넌트가 서로 통신할 수 있도록 HTTP API를 제공한다. 이 Kubernetes API를 사용해서, K8S의 API 오브젝트(예: 파드(Pod), 네임스페이스(Namespace), 컨피그맵(ConfigMap) 그리고 이벤트(Event))를 질의(query)하고 조작할 수 있다. 

[https://kubernetes.io/ko/docs/concepts/overview/kubernetes-api/](https://kubernetes.io/ko/docs/concepts/overview/kubernetes-api/)

### Kubectl의 인증 정보 구조 등장인물. Who?

**K8S Objects related to K8S authentication**

K8S에서 인증정보를 관리하는데는 크게 3가지의 K8S 오브젝트가 동원된다. `User(Service Account), Role, Rolebinding` 이렇게 세가지. 

[K8S Objects related to K8S authentication](https://www.notion.so/b0e66c3e3769456b80409e496acdb555)

![%E1%84%8F%E1%85%AE%E1%84%87%E1%85%A5%E1%84%82%E1%85%A6%E1%84%90%E1%85%B5%E1%84%89%E1%85%B3%20Service%20Account%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20Authenticat%209a27b056adda48be9ea8d3d3d98427d4/Untitled.png](%E1%84%8F%E1%85%AE%E1%84%87%E1%85%A5%E1%84%82%E1%85%A6%E1%84%90%E1%85%B5%E1%84%89%E1%85%B3%20Service%20Account%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20Authenticat%209a27b056adda48be9ea8d3d3d98427d4/Untitled.png)

근데 여기서 알아야 할 점. K8S 사용자의 2가지 컨셉이다. 

### **쿠버네티스 사용자의 2가지 컨셉 (service account와 normal users)**

이와 관련해서 K8S는 normal user account를 직접적으로 관리하는 object 가 없고, normal user account는 API call 을 통해 cluster로 포함될 수 없다. 일반 사용자는 API 호출을 통해 추가할 수 없지만, 클러스터의 인증 기관 (CA) 에서 서명한 유효한 인증서를 제시하는 모든 사용자는 인증된것으로 간주한다. [https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)

user account를 관리하는 object가 왜 없을까.. 왜.. 왜.... 왜 없냐구... 짜증나 진짜..

그래서 K8S에서 권한 요청할 때 대체 누가 해? 라는 질문의 답은 이거다.

API에 인증을 요청할 때는 반드시 normal user, service account, anonymous requests 세 가지 중 하나로 묶여서 인증 절차를 거친 후에 kubernetes api로부터 응답을 받게 된다. 

**KubeConfig**

kubectl은 kubeconfig 파일을 사용하여 클러스터, 사용자, 네임스페이스와 인증 메커니즘에 대한 정보를 관리한다. 3개의 하위 필드로 이루어져 있다. 

![Untitled](https://user-images.githubusercontent.com/25656426/110238697-67474000-7f86-11eb-9f1b-e7a330e50876.png)

### AWS IAM 연결형 K8S 인증 작동 구조

IAM 연결형 K8S 인증 작동 구조는 다음과 같다.

![Untitled 1](https://user-images.githubusercontent.com/25656426/110238680-4ed72580-7f86-11eb-817e-2c071bd3263b.png)


[https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-user-role.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-user-role.html) 

참고로, Amazon EKS 클러스터를 생성한 경우, 클러스터를 생성하는 IAM 엔터티 사용자나 역할에는 클러스터의 RBAC 구성에서 system:masters 권한이 자동으로 부여된다. 

그럼 왜 지금 사용하는 컨텍스트는 아무 권한 준게 없는데 모든 namespace를 조회하냐는 질문을 옆에서 해주셨는데 몰라서 답을 못함..ㅎ 바로 저 위에 시스템 마스터 권한을 자동으로 부여해주는 aws eks 구조 때문이었다. ..

### 마무리하며

K8S의 권한관리는 사실은 그냥 AWS IAM처럼, 권한관리의 기본 of 기본인 Role-based authentication managing을 K8S cluster의 namespace 별로 접근제어하는 것 뿐이다!

namespace 별 접근 제어는 다음과 같은 상황에 유용할 것 같다.

- 클러스터 1개를 사용하는 경우, 타 팀에 관련된 namespace access를 막고 싶을 때
- 내부 개발용(또는 개발 중인 서비스용) namespace 와, DevOps 운영(ex. spinnaker.. 등) 에 관련된 자원들이 한 cluster에 있을 때.

읽어볼만한 링크

[https://kubernetes.io/ko/docs/concepts/overview/kubernetes-api/](https://kubernetes.io/ko/docs/concepts/overview/kubernetes-api/)

[https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-user-role.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-user-role.html)