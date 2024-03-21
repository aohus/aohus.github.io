---
layout: post
title: "[Virtumall #1] 부하테스트 중 일어난 일(1) - nginx worker_connections are not enough"
subtitle:
categories: 프로젝트 실험
tags: ["Virtumall"]
---


웹 서버 구축 및 관리는 복잡한 과제로, 여러 설정과 시스템 리소스의 최적화를 요구합니다. 최근 저는 Nginx를 로드밸런서로 사용하던 중 `worker_connections` 부족으로 인한 에러를 만났습니다.  
```  
1024 working_connections are not enough, reusing connections  
```  

`worker_connections`는 Nginx가 처리할 수 있는 동시 연결의 최대 수를 결정합니다. 이 값이 너무 낮으면 요청을 충분히 처리할 수 없게 되며, 결과적으로 웹 서버의 성능에 직접적인 영향을 미칩니다. 초기 대응으로 `worker_connections`의 수를 증가시키는 것이 일반적인 해결책이나, 이 경우에는 문제 해결에 충분하지 않았습니다. 문제의 근본 원인은 시스템 레벨에서 한 프로세스가 동시에 열 수 있는 파일 디스크립터의 최대 개수(OPEN_MAX)에 있었습니다. 많은 시스템의 `OPEN_MAX` 디폴트 값은 1024로 설정되어 있습니다. 이는 Nginx 프로세스가 동시에 열 수 있는 연결 소켓의 수를 제한하며, 따라서 `worker_connections` 값을 증가시키더라도 시스템 레벨의 제한에 의해 추가적인 연결을 생성할 수 없게 됩니다.  

이 문제를 해결하기 위해서는 시스템의 파일 디스크립터 제한을 증가시켜야 합니다. 이는 운영 체제의 설정을 조정하여 이루어지며, 적절한 값을 설정함으로써 웹 서버의 동시 연결 용량을 크게 향상시킬 수 있습니다.  

#### 파일 디스크립터 제한 증가  
1. 텍스트 에디터를 사용하여 `/etc/security/limits.conf` 파일을 엽니다.  
```bash
sudo vi /etc/security/limits.conf
```  

2. 파일 끝에 다음 라인을 추가하여 파일 디스크립터의 제한을 설정하고 저장합니다.  
```conf  
<사용자_이름> soft nofile <새로운_소프트_리미트>  
<사용자_이름> hard nofile <새로운_하드_리미트>  
```  
  
예를 들어, 모든 사용자에 대해 소프트 리미트를 10240, 하드 리미트를 20480으로 설정하려면 다음과 같이 입력합니다:  
```conf  
* soft nofile 10240  
* hard nofile 20480  
```

3. 변경 사항을 적용하기 위해 시스템을 재부팅하거나 해당 사용자로 로그아웃했다가 다시 로그인합니다.  


#### nginx 설정 변경  
1. `nginx.conf` 파일의 worker_connections 옵션을 변경해줍니다.  
```conf  
worker_connections <max_connection>  
```  

2. 변경 사항을 적용하기 위해 nginx를 재실행해줍니다.  

## 결론  
모두 적용이 되었다면 nginx는 더 많은 사용자의 요청을 처리할 수 있습니다.  