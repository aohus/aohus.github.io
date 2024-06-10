---
layout: post
title: "[DRF] Django Rest Framework란?"
subtitle:
categories: Django DRF
tags: []
---

**Django안에서 Restful API 서버를 쉽게 구축할 수 있도록 도와주는 오픈소스 라이브러리**    
(_REST: HTTP의 URL과 HTTP method(GET, POST, PUT, DELETE)를 사용하여 API 사용 가독성을 높인 구조화된 시스템 아키텍쳐(프레임워크)_)  
  
스마트폰의 탄생 이전 많은 IT 기업들은 웹 페이지를 사용자에게 렌더하기 위한 웹 서버만을 필요로 했다. 그 웹 서버에서 DB 서버의 데이터를 읽어오거나(GET), DB 서버에 데이터를 저장, 수정, 삭제하는(POST, PUT, DELETE) 기능을 모두   담당했다. 하지만 스마트폰 출시 이후 어플리케이션의 등장으로 더 이상 웹으로만은 서비스를 제공할 수 없었다. 즉, HTML로 렌더링하는 웹 서버가 아닌, **JSON 혹은 XML과 같은 형식을 통해 데이터를 다루는 별도의 API 서버가 필요**해진 것이다. (생각해보면 웹이나 앱이나 제공하는 기능과 주고받는 데이터가 동일한데 기존의 웹 서버만을 사용하여 매번 HTML을 읽고 --> 해당 태그에 들어있는 정보를 찾아내어 --> 스마트폰 화면에 띄우는 절차를 진행하는 것은 상당히 비효율적인 일이다.)
  
따라서 RESTful한 아키텍쳐를 HTTP method와 함께 사용하여 웹, 앱, 데스크탑, 스마트폰 어플들까지 범용 가능한 하나의 API를 선호하게 되었다.    
자체 View 클래스 자체가 RESTful한 서버를 만들기에 최적의 프레임워크인 Django 역시 그 기능을 제공하게 된 것이 DRF인 것.  
  
```null  
DRF의 장점 및 DRF를 사용하면 좋은 경우  
  
- 범용성이 좋은 웹 브라우저 API를 사용한 쉬운 개발 ( == RESTful한 서버를 보다 쉽고 빠르게 만들자!)  
- OAuth1, OAuth2를 위한 추가적인 패키지가 인증 정책에 추가되어 있는 경우  
- DB data를 Json으로 직렬화하는(serialize)기능  
- 국제적인 기업들을 포함해서 다수의 기업이 사용 --> 커뮤니티가 잘 되어 있음  
```  
  
## Reference
https://velog.io/@ifyouseeksoomi/DRF-Django-REST-Framework-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%98%88%EC%8A%B5-Serializer  