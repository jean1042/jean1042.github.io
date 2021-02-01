---
date: 2021-02-01
title: "OCI DB services"
categories: 
  - OCI
tags:
  - Cloud services
  - Oracle Cloud Infra
---

# OCI DB service

오늘도 여러가지 클라우드 벤더사 별 서비스 세미나 시간이 있었다. 오늘은 OCI DB 서비스들 발표해주신 SH님이랑, Ali Cloud 맡으신 sofia님께서 제공해주신 소중한 지식 공유. 오늘 OCI 포스팅은 SH님 발표를 토대로 작성했다. All copyrights to Oracle documents and SH..

Oracle Cloud Infra에는 크게 3가지 유형의 DB 시스템이 있다.

헷갈리는 용어가 있어서 이것부터 정의해봤다. 

- **OCI에서의 DB시스템이란?**

    데이터 + DBMS(데이터베이스와 최종 사용자 / 프로그램 간의 인터페이스 역할을 해서 사용자가 정보 구성할 수 있도록 하는 프로그램)+ 응용프로그램

- **Database Edition**이란?

    데이터베이스 엔진 + 배포 옵션이 명시된 오라클 DB의 스펙

- OCI에서의 **Shape**란?

    DB system에 할당될 리소스를 결정하는 스펙

### DB 시스템을 제공하는 node type에 따른 분류

OCI가 시스템을 제공하는 노드유형에 따라 3가지 유형으로 나뉜다. VM / BM / Exadata 의 3가지 DB system.

### VM DB system (가상머신 DB 시스템)

Virtual Machine 위에서 DB가 돌아가는 유형이다.  

2가지 유형의 DB system이 있다. 

- 1 node vm DB system : 1개의 vm 으로 구성된다.
- 2 node vm DB System : 2개의 vm 으로 구성된다.

또 다른 DB system 유형인 `BM(Bare Metal)` 과 사용하는 스토리지 유형, 그리고 scaling에 있어서 차이점이 있다. 

- Storage Type

    VM은 Block storage를 사용하고, BM은 Local NVMe 디스크를 사용한다.

- Scaling

    VM은 storage를 스케일링하고(cpu core수 추후에 변경 못함) 

    BM은 cpu를 스케일링하는 구조 (storage 추후에 변경 X)

### BM DB system (베어메탈 DB 시스템)

Bare Metal DB system은 로컬로 연결된 nvme 스토리지랑 함께, Oracle Linux 6.8에서 구동된다. only 단일 노드 DB시스템을 제공한다(node가 실패하면, 다른 시스템을 시작하고 현재 백업에서 데이터베이스를 복원할 수 있음). 

베어메탈 DB 시스템을 시작할 때, 해당 DB시스템에 적용되는 리소스를 결정하는 Shape를 결정할 수 있다.

서울리전에서는 다음 2개 사용가능! 

![oci_shapes](https://user-images.githubusercontent.com/25656426/106433499-a67c0000-64b3-11eb-82d8-f50af7d3f8f6.png)

note) 데이터베이스 리소스에 대한 초당 청구

VM, BM 인프라를 사용하는 데이터베이스의 경우 Oracle Cloud Infra는 초당 청구를 사용한다. VM DB system의 최소 사용 기간은 1분 / BM은 1시간이다. 발표하신 분이 최소 용량으로 한 3일 켜놨는데 벌써 5만원 넘게 나갔다고 하셨던걸 보니 아주 비싼듯

### Exadata

오라클 자체에서 만든 machine인 Exadata에서 돌아가는 DB system이다. 

storage는 스케일링 못하지만, cpu는 1/4, 1/2 만큼씩 스케일링 가능하고, multiple homes / databases를 지원한다. 

### Database Editions and Versions

DB system을 시작할 때, 해당 DB system에 일괄적으로 적용되는 단일 oracle database version을 선택한다. 

그리고, 지원하는 Oracle Database edition은 DB system 별로 다르다. 

- Single node VM
    - Standard Edition
    - Enterprise Edition
    - Enterprise Edition - High Performance / Extreme Performance

- Two nodes VM Oracle RAC DB 시스템

     Oracle Enterprise Edition - Extreme performance가 최소로 필요하다.

    - Oracle 데이터베이스 21c
    - Oracle 데이터베이스 19c
    - Oracle Database 18c (18.0)
    - Oracle Database *12c* 릴리스 2 (12.2)
    - Oracle Database *12c* 릴리스 1 (12.1)
    - Oracle Database *11g* 릴리스 2 (11.2) 라고 함.

### 기타 특징

OCI console에서 리소스를 생성할 때의 기타 특징은 다음과 같은 것들이 있다.

- Logical Volume Manager를 사용해 관리할 수 있다.
- Oracle의 기존 owned-licence를 가져와서 사용 여부를 결정할 수 있다. 불러오기 가능.
- Database 생성 시 리소스들이 allocated 될 Network 정보도 함께 설정한다.
- Oracle Database Version 선택 시 11/2~ 21C(2021년 2월1일 기준 겁나 최신) 까지의 오라클 데이터베이스 이미지를 사용 가능하다.
- 온프렘에서 oracle을 깔 때 해줬던 sysDB의 설정을 클라우드 콘솔에서 미리 리소스 배포 전 설정할 수 있다.
- 백업 얼마만에 한번 할껀지, Back up scheduling을 설정할 수 있다.
- 기존 on-prem에서 있는 Oracle Provided Database Software Images 가 있다.