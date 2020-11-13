# Front Matter
---
date: 2020-11-39
title: "Prometheus Study"
categories: blog
tags: jekyll
# 목차
toc: true  
toc_sticky: true 
---

# Prometheus

모니터링 도구로 최근 가장 주목받는 것

Soundcloud에서 처음 개발해 현재 CNCF재단에서 관리하는 프로젝트

![Prometheus%206d09cd3cd79945e6975b311b44614ffc/Untitled.png](Prometheus%206d09cd3cd79945e6975b311b44614ffc/Untitled.png)

## 주요 기능

시계열 데이터를 저장할 수 있는 다차원의 모델과 그 모델을 활용할 수 있는 promQL

kube-prometheus-stack

Install 목록

- kubernetes manifests 콜렉션
- 그라파나 대시보드
- prometheus
- Prometheus operator를 포함함

Prometheus operator

- 핵심 아이디어는 모니터링 중인 엔터티의 구성에서 prometheus instance를 분리하는 것.
- 2개의 third party resource가 정의되어 있음 / prometheus / servicemonitor
- operator는 클러스터의 각 리소스에 대해서 원하는 구성을 가진 prometheus server set가 실행되고 있는지 항상 확인함 (데이터 보존 시간, 영구 볼륨 클레임, 복제본 수... 매트릭에 대해 스크랩 할 모니터링 대상 및 매개 변수를 지정)
- manually 지정할 수도 있고, operater 가 second TPR인 servicemontor 가 알아서 하도록 할 수 있음.
- 관리되는 Prometheus 서버를 성공적으로 만들었습니다. 그러나 구성을 제공하지 않았으므로 아직 아무것도 모니터링하지 않습니다. 각 Prometheus 배포는 자체 이름을 딴 Kubernetes ConfigMap을 마운트 합니다. 즉, Prometheus 서버는 네임 스페이스의 "prometheus-k8s"ConfigMap에 제공된 구성을 마운트합니다.

Prometheus operator 설치 시 

- The *prometheus-operator* pod, the core of the stack, in charge of managing other deployments like Prometheus servers or Alertmanager servers
- A *node-exporter* pod per physical host (3 in this example)
- A *kube-state-metrics* exporter
- The default Prometheus server deployment *prometheus-k8s* (replicas: 2)
- The default Alertmanager deployment *alertmanager-main* (replicas: 3)
- The Grafana deployment *grafana* (replicas: 1)]

## Prometheus operator 가 cluster 내 endpoint 를 알아내는 방식

there are two custom resources involved in this process:

- The Prometheus CRD
    - Defines Prometheus server pod metadata
    - Defines # of Prometheus server replicas
    - Defines Alertmanager(s) endpoint to send triggered alert rules
    - Defines labels and namespace filters for the ServiceMonitor CRDs that will be applied by this Prometheus server deployment
        - The ServiceMonitor objects will provide the dynamic target endpoint configuration
- The ServiceMonitor CRD
    - Filters endpoints by namespace, labels, etc
    - Defines the different scraping ports
    - Defines all the additional scraping parameters like scraping interval, protocol to use, TLS credentials, re-labeling policies, etc.

    안되는거

    172-16-17-216

    172-16-16-217

    172.16.16.25

    ## Prometheus가 label_values를 알아내는 방법

    label_values(kube_pod_info{cluster="$cluster"}, node) 안되는거

    label_values(node_uname_info{}, instance)  

    prometheus의 다양한 메트릭 정보를 수집해서 /metrics라는 엔드포인트로 제공해줌. 

    쿠버네티스 클러스터의 마스터 노드에 존재하는 kubernetes api와 통신하여 메트릭 정보를 수집하기 때문에 1개의 파드... 

    ## Prometheus를 이용하여 모니터링 하는 영역

    - 클러스터에 대한 정보 : kube-state-metrics로 수집함
    - 노드에 대한 정보 : node exporter로 수집함
    - 리소스 사용량에 대한 정보 : node, pod 등의 리소스 사용량을 수집함.
    - 직접 만든 서비스에 대한 정보

## Prometheus Operator

- install and provide same initial configuration and sizing for my deployment, according to the specs of my kuber cluster

관련 document

[https://coreos.com/blog/the-prometheus-operator.html](https://coreos.com/blog/the-prometheus-operator.html)

[https://sysdig.com/blog/kubernetes-monitoring-prometheus-operator-part3/](https://sysdig.com/blog/kubernetes-monitoring-prometheus-operator-part3/)

design document [https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/design.md](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/design.md)