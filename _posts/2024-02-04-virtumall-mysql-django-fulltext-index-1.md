---
layout: post
title: "[Virtumall #3] MySQL FULLTEXT 인덱스란?"
subtitle:
categories: 프로젝트
tags: ["Project Virtumall"]
---
  
## 기존 검색  
SQL 쿼리에서 검색 성능을 높이기위해 인덱스의 사용은 매우 중요합니다. 인덱스는 보통 `B-Tree` 구조로 가장 왼쪽 접두사를 사용하는 경우에만 유용합니다.  

`Virtumall` 프로젝트에는 `product`(상품) 테이블이 있고 사용자가 상품을 검색하면 `product.name`에 검색어가 포함되어 있는지 확인하여 결과를 반환했습니다.  
`product.name`에 인덱스가 적용되어 있었지만, `SELECT * FROM product WHERE product.name LIKE '%keyword%'` 와 같이 검색을 하고 었고 LIKE %keyword% 구문은 인덱스를 타지않아 full scan이 이루어지고 있었습니다. 
![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-02-04-fulltext-01.png?raw=true)  
- 아래 `type: ALL`, `key: NULL`, `rows: 231983`으로 full scan 하고 있다는 것을 볼 수 있습니다.  

  
실제 결과를 반환하는데는 0.55 sec 소요되었습니다.   
```sql  
SELECT * FROM product   
WHERE product.name LIKE '%good%'   
ORDER BY product.created_at;  
```  
  
![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-02-04-fulltext-02.png?raw=true)  
  
## FULLTEXT 인덱스  
FULLTEXT는 값을 인덱스의 값과 직접 비교하는 대신 택스트에서 키워드를 찾는 특수 유형의 인덱스입니다. 풀 텍스트 검색은 다른 유형의 일치 검색과 다릅니다. 단순한 WHERE 매겨변수 매칭보다 검색 엔진이 하는 작업과 훨씬 더 유사합니다.  FULLTEXT 인덱스는 인덱스에서 지원하기 어려운 검색 조건을 지원하기 위해 특정 컬럼의 데이터를 Parsing 하여 각 단어가 속한 문서의 ID를 저장하는 방식으로 구현되었습니다. "롯데 간편식 고등어"라는 TEXT 데이터를 Parsing하여 어절별로("롯데", "간편식", "고등어") 각 단어가 속한 record의 ID를 저장하는 것입니다. 따라서 `LIKE '%keyword%'` 검색보다 훨씬 더 성능이 좋으며, 자연어 검색이나 불린 검색 등 다양한 검색 모드를 지원합니다.  
  
### MYSQL에서 FULLTEXT를 설정  
```sql  
-- 1) 테이블 생성할 때
CREATE TABLE 테이블명(
    컬럼명 TEXT DEFAULT NULL, # 역인덱싱 대상 컬럼
    ...,
	FULLTEXT 인덱스명(컬럼명) #FULLTEXT 인덱스명(대상 컬럼명)
)

-- 2) 이미 생성된 테이블에 대해
CREATE FULLTEXT INDEX 인덱스명 ON 테이블명 (컬럼명);
ALTER TABLE 테이블명 ADD FULLTEXT(컬럼명);  
```  
  
몇가지 고려해야할 점은 아래와 같습니다.   
- **스토리지 엔진**: FULLTEXT 인덱스는 InnoDB 및 MyISAM 스토리지 엔진에서 지원됩니다. 사용 중인 MySQL의 버전과 테이블의 스토리지 엔진을 확인하세요. MySQL 5.6 이상에서는 InnoDB에서도 FULLTEXT 인덱스를 지원합니다.  
- **성능 영향**: FULLTEXT 인덱스를 추가하면 해당 컬럼에 대한 쓰기 연산(INSERT, UPDATE)이 더 느려질 수 있습니다. 인덱스 생성 과정도 시간이 다소 소요될 수 있으므로, 운영 환경에서는 적절한 시간에 작업을 수행해야 합니다.  
- **검색 기능**: FULLTEXT 인덱스를 사용하면, `MATCH() ... AGAINST()` 구문을 사용하여 텍스트 검색 쿼리를 실행할 수 있습니다.   
- **인덱스 관리**: 인덱스가 생성된 후에는, 정기적인 유지보수 및 최적화 작업을 고려해야 할 수 있습니다. 예를 들어, `OPTIMIZE TABLE` 명령을 사용하여 인덱스를 재구성하고, 저장 공간을 최적화할 수 있습니다.  
  
따라서 조회는 자주 되지만 비교적 INSERT, UPDATE는 적은 컬럼에 대해 사용해야합니다. 또한 그에 따른 시스템 리소스 사용 증가와 유지 관리 요구 사항을 고려해야 합니다.  
  
  
### MATCH ~ AGAINST 구문으로 검색  
FULLTEXT INDEX가 사용된 컬럼에 대한 검색은 `WHERE MATCH(column_name) AGAINST('key1 key2' IN NATURAL LANGUAGE MODE)`와 같은 구문으로 검색합니다.   

검색 모드 옵션은 다음과 같이 사용할 수 있습니다.  
- **NATURAL LANGUAGE MODE**: 가장 일반적인 검색 모드로, 검색어가 자연스러운 언어의 일부인 것처럼 검색합니다.  
- **BOOLEAN MODE**: 검색어에 특수한 불린 연산자(+, -, 등)를 사용할 수 있습니다. 이 모드를 사용하면 더 세밀한 검색 제어가 가능합니다.  
- **WITH QUERY EXPANSION**: 검색 결과를 기반으로 검색어를 확장하여, 관련된 추가 결과를 찾습니다. 이는 초기 검색 결과가 적을 때 유용할 수 있습니다.  

#### NATURAL LANGUAGE MODE
```sql
SELECT * FROM product
WHERE MATCH(name) AGAINST('good your' IN NATURAL LANGUAGE MODE)
ORDER BY product.created_at;
```
- 해당 모드는 검색 문자열을 단어 단위로 분리한 후, 해당 단어가 포함되는 문서를 찾아서 반환합니다.
- name(상품명)에 good 혹은 your이 포함된 데이터를 반환합니다.

#### BOOLEAN MODE
```sql  
-- name(상품명)에 good과 your이 모두 포함된 데이터를 반환합니다.  
SELECT * FROM product  
WHERE MATCH(name) AGAINST('+good +your' IN BOOLEAN MODE)  
  
-- name(상품명)에 good이 포함된 데이터를 반환하며, 가능한 your가 함께 포함된 데이터를 우선 반환합니다.  
SELECT * FROM product  
WHERE MATCH(name) AGAINST('+good your' IN BOOLEAN MODE)  
```

#### WITH QUERY EXPANSION  
- 2단계에 거쳐 검색을 수행하며, 1단계에서 자연어 검색을 수행한 이후 첫번째 검색 결과를 기반으로 관련성이 높은 데이터들을 선별한 다음 검색 문자열을 재구성하여 2번째 검색을 수행합니다.  
```sql   
SELECT * FROM product  
WHERE MATCH(name) AGAINST('good*' WITH QUERY EXPANSION)  
```  

## 결과  
FULLTEXT 인덱스 설정 후 MATCH ~ AGAINST 로 검색 시 `type: fulltext`, `key: name` 로 바뀐 것을 볼 수 있습니다.  
![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-02-04-fulltext-03.png?raw=true)  
  
1트: 0.32 sec  
![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-02-04-fulltext-04.png?raw=true)  
  
2트: 0.00 sec  
- FULLTEXT 인덱스는 첫 번째 검색 시 인덱스를 메모리로 로딩하는데 시간이 소요될 수 있으나, 일단 메모리에 로딩되면 후속 검색에서는 이 인덱스를 빠르게 액세스합니다. FULLTEXT 인덱스의 경우, 검색 결과는 내부적인 캐시에 저장될 수 있으며, 특히 BOOLEAN MODE에서는 쿼리의 결과를 재사용할 수 있어 두 번째 검색이 더 빨라질 수 있습니다. 이는 인기 검색어에 대한 쿼리 수행에 적합합니다. 또한, MySQL은 FULLTEXT 검색의 중간 처리 단계(예: 검색어 파싱, 랭킹 계산)에서 얻어진 정보를 메모리에 캐싱할 수 있으며, 이 정보는 후속 검색에서 재사용될 수 있어 처리 시간을 단축시킵니다.
- MySQL 8.0 이상에서는 쿼리 캐시 기능이 제거되었기 때문에 인덱스를 활용하지 않는 LIKE %% 구문 같은 경우는 동일한 키워드 검색에도 매번 풀스캔이 일어납니다.  
![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-02-04-fulltext-05.png?raw=true)  
  
  
  
## References  
<https://docs.djangoproject.com/en/5.0/howto/custom-lookups/>  