---
layout: post
title: "[Django] Django 에서 db connection 관리"
subtitle:
categories: Django database
tags: []
---

서비스를 운영하다보면 DB Connection 관리에 대한 이슈가 종종 발생합니다. 이번 포스팅에서는 Django에서 DB 커넥션을 어떻게 관리하는지 알아보겠습니다.

기본적으로 Django는 내장된 Connection Pooling 을 제공하지 않습니다. Django의 ORM은 각 요청에 대해 데이터베이스 연결을 열고 요청이 완료되면 연결을 닫습니다. 즉, Django는 "long-lived" 연결을 유지하지 않고, 연결을 필요할 때마다 생성하고 해제한다는 것입니다. 그렇다고 연결 오버헤드를 방치하는 것은 아닙니다. Persistent Connection 방식을 적용하여 DB 쿼리를 처음 만들 때 DB 커넥션을 열고 이 연결을 열린 상태로 유지하고 후속 요청에서 이 연결을 재사용합니다.

### Django 의 db connection 최적화
Django의 db connection을 최적화 하는 방식은 크게 두 가지가 있습니다. Django에 내장된 Persistent Connection 방식을 사용하는 것과 외부 라이브러리를 통해 [[connection pool|Connection Pooling]]을 사용하는 것입니다. 

1. **persistent connection**
	Django의 `persistent-db-connections` 설정을 활성화하여 Django의 기본 데이터베이스 연결을 재사용할 수도 있습니다. 이는 Django 1.6부터 도입된 기능입니다. `persistent connection`은 전통적인 Connection Pooling과는 다르며, `CONN_MAX_AGE` 커넥션의 재사용 시간을 결정하는 것입니다.

2. **connection pooling**
	(connection pool에 대한 내용은 다음 포스팅을 참고해주세요! >> [connection pool 이란?]()  )
	Django에서 [[connection pool|Connection Pooling]]을 사용하려면, 외부 라이브러리를 활용해야 합니다. `django-db-connection-pool`은 Django 애플리케이션에 대해 Connection Pooling을 제공하는 라이브러리입니다. 이 라이브러리는 `SQLAlchemy`의 연결 풀 기능을 사용하여 작동합니다.

### Persistent Connection
Persistent Connection을 사용하려면 `CONN_MAX_AGE`를 설정해야 합니다. 이 값은 DB 커넥션의 재사용 시간이며 따로 설정하지 않으면 모든 요청마다 커넥션을 열고 닫습니다. `CONN_MAX_AGE`는 초 단위이며 정수 값으로 설정할 수 있습니다. 무제한으로 열린 상태를 유지하고 싶으면 `None`으로 설정합니다. 

```
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'dbname',
    'USER': 'dbuser',
    'PASSWORD': 'password',
    'HOST': '127.0.0.1',
    'PORT': '3306',
    'CONN_MAX_AGE': 60,
  }
}
```

## References
[Django에서 DB Connection 관리](https://seungho-jeong.github.io/technology/computer-science/django-db-connections/)
[DB Connection을 관리하는 방법](https://americanopeople.tistory.com/260)
https://docs.djangoproject.com/en/5.0/ref/databases/#persistent-database-connections
