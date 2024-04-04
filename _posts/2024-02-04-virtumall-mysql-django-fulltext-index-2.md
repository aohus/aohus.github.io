---
layout: post
title: "[Virtumall #4] Django에 MySQL FULLTEXT 인덱스 적용하기(feat. custom-lookups)"
subtitle:
categories: 프로젝트
tags: ["Project Virtumall"]
---
  
FULLTEXT 인덱스가 무엇이고 언제 사용하면 좋은지, 명령어는 어떻게 사용하는지를 보려면 [MySQL FULLTEXT 인덱스란?](https://aohus.github.io/프로젝트/2024/02/04/virtumall-mysql-django-fulltext-index-1.html)을 참고해주세요. 이 포스팅에서는 FULLTEXT 인덱스를 Django 어플리케이션에 적용하는 방법을 설명합니다.   
  
  
## Django에 FULLTEXT 인덱스 적용하기  
먼저 현재 장고에서는 FULLTEXT 인덱스를 ORM으로 지원하지 않기 때문에 직접 데이터베이스에 FULLTEXT 인덱스를 적용하는 쿼리를 해줘야합니다.   
  
#### 1. 아래 명령어로 마이그레이션 파일을 직접 생성해줍니다.   
  
```shell  
python manage.py makemigrations {app_name} --empty --settings=app.settings.base  
```  
  
#### 2. 생성된 마이그레이션 파일에는 다음과 가타은 SQL 명령을 추가하여 직접 `FULLTEXT INDEX`를 생성해줍니다.   
  
```python  
from django.db import migrations  
  
  
class Migration(migrations.Migration):  
  
    dependencies = []  
  
    operations = [  
        migrations.RunSQL(  
            ("CREATE FULLTEXT INDEX 인덱스명 ON 테이블명(컬럼명)",), # 인덱스 생성 명령  
            ("DROP INDEX 인덱스명 ON 테이블명",), # 마이그레이션 롤백시 인덱스 삭제해줄 명령  
        ),  
    ]  
  
```  
  
#### 3. [커스텀 Lookup](https://docs.djangoproject.com/en/5.0/howto/custom-lookups/)을 생성하여 FULLTEXT 인덱스를 사용하는 쿼리를 하도록 설정합니다.  
아래 파일을 원하는 위치에 생성해줍니다.(저는 `utils` 디렉터리의 `lookup.py`로 생성했습니다.)  
```python  
from django.db.models import Field  
from django.db.models import Lookup  
  
  
@Field.register_lookup #search(lookup_name) 키워드를 사용하여 쿼리할 수 있도록 등록  
class SearchLookup(Lookup):  
    lookup_name = "search" # column__icontains의 icontains와 같은 키워드를 설정  
  
    def as_mysql(self, compiler, connection):  
        lhs, lhs_params = self.process_lhs(compiler, connection)  
        rhs, rhs_params = self.process_rhs(compiler, connection)  
        params = lhs_params + rhs_params  
        return "MATCH (%s) AGAINST (+%s IN BOOLEAN MODE)" % (lhs, rhs), params  
```  
  
#### 4. 마지막으로 `settings` 파일에 `lookups.py` 파일을 포함시켜줍니다.   
```python  
from utils.lookups import *  
```  
  
## FULLTEXT 인덱스 사용하여 검색하기  
위에서 등록해준 `loopup_name`인 `search`는 다음과 같이 사용됩니다.   
다음은 `product`.`name` 에서 `참치`가 포함된 데이터를 검색하는 법입니다.  
  
```python  
Product.objects.filter(name__search='참치')   
```  
  
위의 예에서 process_lhs는 ('"product"."name"', [])을 반환하고 process_rhs는 ('"%s"', ['참치'])을 반환합니다.    
따라서 `Product.objects.filter(name__search='참치')`은 SearchLookup에 정의해준대로 다음과 같은 쿼리를 반환합니다.  
  
```sql  
SELECT * FROM product    
WHERE MATCH(name) AGAINST('+참치' IN BOOLEAN MODE)    
```  
  
## References    
<https://docs.djangoproject.com/en/5.0/howto/custom-lookups/>    