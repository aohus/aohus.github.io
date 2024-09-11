---
layout: post
title: "[Kubernetes] Control Plane의 ControllerManager"
subtitle:
categories: Kubernetes
tags: []
---
## 들어가며
쿠버네티스에서 ReplicaSet은 파드(Pod)의 원하는 개수를 항상 유지하는 역할을 하는 컨트롤러입니다. 만약 파드가 중지되거나 실패하면 ReplicaSet이 새로운 파드를 생성해 그 수를 맞추고, 파드 수가 초과되면 여분의 파드를 삭제합니다. ReplicaSet은 **컨트롤러 관리자(controller manager)**에 의해 실행되며, 이는 쿠버네티스 API 서버와 상호작용해 클러스터 상태를 조정합니다.

ReplicaSet의 내부 동작과 파드를 관리하는 방식을 코드 레벨에서 살펴보면, 핵심적인 구성 요소와 로직이 쿠버네티스 코드베이스의 kubernetes/kubernetes 저장소 내에 있습니다. 주요 파일은 pkg/controller/replicaset 디렉토리에 있으며, ReplicaSet의 구현체는 Go 언어로 작성되어 있습니다.

## 전체 그림
![2024-09-06-kubernetes-controller-manager.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-09-06-kubernetes-controller-manager-tmp.png?raw=true)


## Reference  
