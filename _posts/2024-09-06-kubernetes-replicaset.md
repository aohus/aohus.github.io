---
layout: post
title: "[Kubernetes] 쿠네티스 ReplicaSet"
subtitle:
categories: Kubernetes
tags: []
---
## 들어가며
쿠버네티스에서 ReplicaSet은 파드(Pod)의 원하는 개수를 항상 유지하는 역할을 하는 컨트롤러입니다. 만약 파드가 중지되거나 실패하면 ReplicaSet이 새로운 파드를 생성해 그 수를 맞추고, 파드 수가 초과되면 여분의 파드를 삭제합니다. ReplicaSet은 **컨트롤러 관리자(controller manager)**에 의해 실행되며, 이는 쿠버네티스 API 서버와 상호작용해 클러스터 상태를 조정합니다.

ReplicaSet의 내부 동작과 파드를 관리하는 방식을 코드 레벨에서 살펴보면, 핵심적인 구성 요소와 로직이 쿠버네티스 코드베이스의 kubernetes/kubernetes 저장소 내에 있습니다. 주요 파일은 pkg/controller/replicaset 디렉토리에 있으며, ReplicaSet의 구현체는 Go 언어로 작성되어 있습니다.

1. ReplicaSet 컨트롤러 개요
ReplicaSet의 핵심적인 컨트롤러는 controller manager에 의해 실행되며, ReplicaSet의 상태를 주기적으로 모니터링하고, 주어진 스펙과 실제 파드 상태를 비교해 차이를 보정합니다. 그 핵심 로직은 아래와 같은 작업을 수행합니다:

현재 클러스터에 존재하는 파드의 수를 확인
ReplicaSet의 spec.replicas에서 설정된 파드 개수와 비교
부족하면 새로운 파드를 생성하고, 초과하면 파드를 삭제


## Reference  
