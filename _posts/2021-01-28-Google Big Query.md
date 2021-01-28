---
date: 2021-01-28
title: "Google Big Query"
categories: 
  - Google Cloud Platform
tags:
  - Cloud services
  - Google Cloud
---
# Google Big Query

각 벤더사 별 클라우드 서비스에 대한 knowledge share를 할 수 있는 mini seminar 기회가 주기적으로 있는 편이다. 학교에서 배우고, 이젠 내 인생의 중요한 철학이 된 Learn to share, share to learn를 자동으로(?) 실현 가능해서 아-주 좋다ㅎㅎ.

그 중에서 오늘은 Google Cloud service를 담당하시는 분의 `Big Query` knowledge share 시간이 있었고, 세미나에서 들은 내용을 간단하게 정리해봤다.

## Google Big Query

- 자체 Query engine이 내장된 serverless 대규모 데이터 저장 및 분석 플랫폼, 데이터웨어 하우스(저장소)
- 대용량 dataset을 분석하는데 사용할 수 있는 웹 서비스

### 그럼.. 대체 이걸 언제 쓰는지?

예를 들어서, 이커머스 에서 쌓인 데이터들을 데이터분석을 돌리고 싶은데, 기존의 DB에서 구글 빅쿼리로 데이터를 밀어넣어서(api, excel, dump.. 등등) 빅쿼리에서 데이터분석용 쿼리를 돌리는 용도로 사용한다. 

### Pros and Cons

**Pros**

- 내장된 쿼리 엔진으로 TB 데이터에 대한 sql쿼리를 단 몇초 만에 실행함 🔺
- 쿼리 Step별 디버깅(?)이 가능한 쿼리의 Plan 단위가 존재한다 (쿼리 실행의 Plan 단위) 🔺
- 에러 검색이 쉬움 (syntax 에러도 쿼리 기록으로 저장) 🔺
- 타사 비용 대비 저렴함 🔺
- 관계형 데이터베이스 구조 🔻

**Cons**

- 스키마 정의 필요 🔻
- DocDB 같은 Non relational은 불가능 🔻
- 분석용 말고 우리같은 작업 DB로 쓰는건 한계가 있음 🔻
    - data set에 대한 정보 만료기간이 있음 🔻
    - 정확한 region 지정 불가 (zone 까지 지정 못함)🔻

### 가격 책정

- 저장공간에 대한 비용 (스토리지 레벨 저장 정책)

    활성 스토리지 + 장기 스토리지 

- Big Query Storage API에 대한 비용 (ex. 내 빅쿼리 리스트에 존재하는 테이블 데이터 list 줘~)
    - job 별 total bytes processed data 를 포함한다.
- 스트리밍 삽입.. 등등의 사용량 기반으로 책정하는 비용
- 시간별 가변 슬롯 약정제 / 월간 정액제 / 연간 정액제 제공

### 특징

- Public Dataset 을 marketplace에서 다운받을 수 있다. 예를 들어, 000지역의 COVID-19 데이터셋, 비트코인 데이터셋.. 등 여러개 데이터셋이 있다.

- **Project**와 **Job**의 개념

    Google Big Query 에는 Project / Job이라는 기존 관계형 DB에 없는 개념이 있다. 

    - Project(구글 클라우드 특유의 리소스 관리 단위) 별 → Data set (RDS 기준으로 cross-연산이 가능한 database정도로 이해하면 될듯) 별 →  Table 순이다.
    - Job은? → 작업 id, 기간, 처리한 바이트, 청구된 바이트, 작업 우선순위 등등이 묶여진 쿼리 하나의 실행 단위
    - Job별로 (쿼리 플랜 info가 포함된) API를 받을 수 있다.

### 발표자 분께서 생각하시는 Google Cloud Big Query의 Key Properties

- 데이터셋 별 (성격이 다른 데이터를 묶어놓은 것들) 통계를 낼 수 있다.

    큰 다른 주제별 (?) 통계에 따른 다른 가치를 뽑아낼 수 있다는 점에서, 발표자님이 좋은 기능이라고 생각하신 듯 하다.

- Reference table, Target table이 나눠져 있다.
- 쿼리 별 Plan이 존재하고, 그 Plan 별로 쿼리 디버깅이 가능하고 / API 호출 시의 단위도 되어서 조금 강조하신 부분.