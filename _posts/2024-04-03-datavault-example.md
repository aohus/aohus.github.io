---
layout: post
title: "[Data Engineering] 데이터 볼트(Data Vault)의 구체적인 예시"
subtitle:
categories: DataEngineering
tags: []
---

## 들어가며
[데이터 모델링2 - 데이터 볼트(Data Vault)를 이해해보자(3NF vs Dimensional model vs Data Vault)]()에서 본 데이터 볼트 모델링 방법으로, 실제로 모델링을 해보려고합니다. 구체적인 쿼리와 함께 살펴봅시다. 

## 데이터 볼트(Data Vault) 소개
데이터 웨어하우징은 데이터를 유용한 정보로 통합, 집계, 요약하기 위한 기술입니다. 이를 위해 데이터 웨어하우스 엔지니어는 스테이징 계층, 클렌징 계층, 코어 계층, 데이터 마트 계층과 같은 계층을 기반으로 다양한 아키텍처를 적용합니다. 코어 계층의 목표는 여러 소스의 데이터를 통합하고 기록하는 것입니다. 또한 코어 계층은 BI 애플리케이션에 사용되는 데이터 마트로 데이터를 전달하는 기능도 가지고 있습니다. 데이터 마트의 데이터 모델은 일반적으로 dimension과 fact가 포함된 파원 데이터 모델이지만, 코어에서는 다양한 모델링 방법을 적용할 수 있습니다. 이러한 데이터 모델링 방법은 다음과 같습니다: 

- dimension과 fact table을 사용한 차원 핵심 모델링
- 원자 데이터의 통합 레파지토리
- 허브, 링크, 새틀라이트가 포함된 데이터 볼트 모델링

각 접근 방식에는 요구 사항에 따라 장점이 있으므로 이러한 접근 방식을 조합하여 사용할 수도 있습니다. 데이터 볼트는 허브, 링크, 새틀라이트의 세 가지 기본 엔티티 유형으로 구성됩니다. 허브는 비즈니스 키를 나머지 모델과 분리하고, 링크는 허브 간의 연결(비즈니스 키)을 저장하며 새틀라이트는 허브 또는 링크의 속성을 저장합니다. 이러한 엔티티의 구조와 엔티티 간의 관계를 설명하기 위해 잘 문서화된 데모 데이터베이스인 Airlines의 3NF 모델을 Data Vault 모델로 변환합니다. 3NF 모델은 Figure1에 나와있습니다. 데모 데이터베이스는 PostgreSQL 라이선스에 따라 배포되며 5에서 사용할 수 있습니다. 여기에는 항공사 항공편에 대한 데이터가 포함되어 있으며 8개의 테이블과 여러개의 보기로 구성되어 있습니다. 

![2024-04-01-datavault-01.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-04-01-datavault-01.png?raw=true)

## 데이터 볼트(Data Vault) 엔티티
Figure1에 있는  `aircrafts_data` 와 `seats` 간의 관계를 모델링하여 데이터 볼트 엔티티를 소개합니다. 
![2024-04-01-datavault-02.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-04-01-datavault-02.png?raw=true)

### The Hub
 허브 엔티티는 비즈니스 개체의 비즈니스 키를 동일한 의미론적 세분성으로 저장합니다. 허브는 다음과 같은 속성으로 구성됩니다. 
 - 비즈니스키는 비즈니스 개체를 식별하는 자연 키를 나타냅니다.
 - 기본키는 비즈니스 키의 해시값 또는 시퀀스 번호(대리 키)입니다. Data Vault2.0에서는 시퀀스 번호 기반 기본키가 해시 기반 기본키로 대체됩니다. 
 - `load_date_ts`는 비즈니스 키가 처음 허브에 도착한 날짜와 시간을 나타냅니다. 
 - Record source 는 비즈니스 키의 source system을 나타냅니다. 이는 도착한 레코드의 추적성 유지를 위한 것입니다. 

비즈니스 키는 비즈니스 개체의 데이터를 식별, 추적 및 위치시킵니다. Figure1의 `Bookings` 스키마에서 비즈니스 개체의 예로 `airport`를 들 수 있는데, 공항 개체는 `airport_code`로 식별됩니다.(즉, `airport`의 비즈니스 키는 `airport_code`입니다.) 비즈니스 개체의 또 다른 예는 비즈니스 키가 `aircraft_code`인 항공기입니다. 비즈니스 키는 변경되는 경향이 적으며 운영 시스템 내에서 고유해야합니다. 

예를 들어, Figure1의 엔티티 `aircrafts_data`의 허브는 다음 SQL문으로 생성된 Figure2의 엔티티 `h_aircraft`입니다. 

```SQL
create table dv.h_aircraft (  
	hk_aircraft char(32) not null  
	,aircraft_code char(3) not null  
	,load_date_ts timestamp not null  
	,constraint h_aircraft_pk  
	primary key (hk_aircraft)  
)
```

따라서 `h_aircraft`의 각 레코드는`aircraft_data` 의 비즈니스 키 `aircraft_code`와 해당 해시 값을 레코드의 기본키로 구성합니다. 따라서 다음 SQL문을 사용하여 `h_aircraft`의 초기 로드를 수행할 수 있습니다. 

```SQL
insert into dv.s_aircraft (  
	hk_aircraft  
	,load_date_ts  
	,model  
	,range  
	) select  
	md5(upper(trim(aircraft_code)))  
	,current_timestamp(0)  
	,model ->> 'en' as model  
	,range  
	from bookings.aircrafts_data
```

해시값을 계산하기 위해 `MD5` 알고리즘을 사용합니다. 또 다른 권장 옵션은 `SHA-1` 알고리즘입니다. `aircraft_code` 값의 오른쪽과 왼쪽에서 빈 공백을 모두 제거하기 위해 trim() 함수를 적용합니다. 그런 다음 upper()함수를 사용하여 `aircraft_code` 값을 표준화합니다. 
Figure1의 스키마에서 엔티티 좌석의 허브는 다음 SQL 문에 의해 생성된 Figure2의 엔티티 `h_seat`입니다.
```SQL
create table dv.h_seat (  
	hk_seat char(32) not null  
	,aircraft_code char(3) not null  
	,seat_no varchar(4) not null  
	,load_date_ts timestamp not null  
	,constraint h_seat_pk  
	primary key (hk_seat)
```

`h_seat`의 비즈니스 키는 `aircraft_code`와 `seat_no`로 구성된 복합 비즈니스 키라는 점에 유의하세요. 다음 명령으로 `h_seat`의 초기 로드를 수행합니다. 

```SQL
insert into dv.h_seat (  
	hk_seat  
	,aircraft_code  
	,seat_no  
	,load_date_ts  
	) select  
	md5(  
		upper(trim(aircraft_code))  
		|| '|'  
		|| upper(trim(seat_no))  
	)  
	,aircraft_code  
	,seat_no  
	,current_timestamp(0)  
	from bookings.seats  
	)
```

허브 엔티티는 규칙을 준수합니다:
- 허브 엔티티는 절대로 외래 키를 포함하지 않습니다. 
- 허브는 독립적으로 존재해야 합니다. 다른 엔티티를 참조하지 않아야합니다. 
- 허브에는 적어도 하나의 위성(`satellite`)가 있어야합니다. 
- 허브의 기본 키와 비즈니스 키는 절대 변경되지 않습니다. 
- 허브 간의 직접적인 관계는 허용되지 않습니다.(허브대 허브)
- 허브는 설명 속성을 포함하지 않습니다. 

### The Link
링크는 허브와 링크를 연결합니다. 링크는 다음 속성으로 구성됩니다. 
- 기본키: 연결된 허브 또는 링크의 모든 비즈니스 키의 해시 값입니다. 
- 외래키: 링크로 연결된 각 허브에 대한 키이며 값은 해당 비즈니스 키의 해시 값입니다. 
- Load date: 링크에 연결이 처음 기록된 날짜와 시간을 저장합니다.
- 레코드 소스: 관계가 생성된 소스 시스템을 저장합니다. 

예를 들어, Airline 스키마에서 `aircraft_data`와 좌석 간에 정의된 연관성을 모델링하는 링크는 Figure2의 `l_aircraft_seat`로 모델링 되며 다음 SQL 문으로 정의됩니다. 

```SQL
create table dv.l_aircraft_seat (  
	lhk_aircraft_seat char(32) not null  
	,hk_aircraft char(32) not null  
	,hk_seat char(32) not null  
	,load_date_ts timestamp not null  
	,constraint l_aircraft_seat_pk  
		primary key (lhk_aircraft_seat)  
	,constraint l_aircraft_seat_h_aircraft_fk  
		foreign key (hk_aircraft)  
		references dv.h_aircraft (hk_aircraft)  
	,constraint l_aircraft_seat_h_seat_fk  
		foreign key (hk_seat)  
		references dv.h_seat (hk_seat)
```

허브 `h_aircraft`와 `h_seat`간의 연결이 해시 키를 참조하여 정의되는 방식에 주목하세요. `l_aircraft_seat`의 초기 로드는 링크의 속성을 이해하는데 도움이됩니다. 

```SQL
insert into dv.l_aircraft_seat (  
	lhk_aircraft_seat  -- pk
	,hk_aircraft       -- fk
	,hk_seat           -- fk
	,load_date_ts  
	) select  
	md5(  
		upper(trim(aircraft_code))     --- dv.h_aircraft의 hk_aircraft(pk)
		|| '|'  
		|| upper(trim(aircraft_code))  --- dv.h_seat의 hk_seat(pk)
		|| '|'                         --|
		|| upper(trim(seat_no))        ---
		)  
	,md5(upper(trim(aircraft_code)))   --- dv.h_aircraft의 hk_aircraft(pk)
	,md5(  
		upper(trim(aircraft_code))     --- dv.h_seat의 hk_seat(pk)
		|| '|'                         --|
		|| upper(trim(seat_no))        ---
		)  
	,current_timestamp(0)  
from bookings.seats
```

보시다시피 링크의 기본 키는 두 값을 연결한 후 해당 값의 해시값을 계산하여 생성됩니다. 참조된 각 비즈니스 키에 대한 외래 키는 해당 비즈니스 키의 해시 값입니다. 

링크 엔티티는 다음과 같은 속성이 있습니다. 
- 링크 엔티티에는 비즈니스 키를 포함할 수 없습니다. 
- 링크는 항상 다대다 관계입니다. 
- 링크 테이블에는 설명 속성이 포함되지 않습니다. 
- 링크에는 새틀라이트가 있을 수 있습니다. 
- 링크는 다른 링크에 연결될 수 있습니다. 

### The Satellite
Satellite는 비즈니스 개체(즉, 허브) 또는 관계(즉, 링크)를 설명하는 모든 속성을 저장합니다. 새틀라이트는 하나의 허브 또는 링크에만 연결됩니다. 따라서 해당 허브 또는 링크의 해시 키와 속성 값의 변경 타임스탬프로 식별됩니다. 새틀라이트의 목적은 비즈니스 개체 또는 비즈니스 개체가 연결된 관계를 설명하는 설명 데이터의 변경 사항을 추적하는 것입니다. 

예를들어 허브 `h_seat`과 위성 `s_seat`은 허브에 속성이 있으므로 좌석의 비즈니스 키를 정의하는 속성을 제외한 테이블 좌석의 모든 속성을 채택합니다. 다음 SQL문은 새틀라이트 `s_seat`을 생성합니다. 

```SQL
create table dv.s_seat (  
  hk_seat         char(32)    not null  
 ,load_date_ts    timestamp   not null  
 ,fare_conditions varchar(10) not null  
 ,constraint s_seat_pk  
    primary key (hk_seat, load_date_ts)  
 ,constraint s_seat_h_seat_fk  
    foreign key (hk_seat)  
    references dv.h_seat(hk_seat)  
)
```

`s_seat`의 기본키가 `load_date`와 허브 `h_seat`의 해시 키의 복합 키라는 점에 유의하세요. `s_seat`의 초기 로딩은 다음과 같이 수행됩니다. 

```SQL
insert into dv.s_seat (  
   hk_seat  
 ,load_date_ts  
 ,fare_conditions  
 ) select  
  md5(  
   upper(trim(aircraft_code))   --- dv.h_seat의 hk_seat(pk)
    || '|'  
    || upper(trim(seat_no))  
   )  
  ,current_timestamp(0)  
  ,fare_conditions   
from bookings.seats
```

데이터 웨어하우스의 기능 중 하나는 데이터의 기록 보기를 제공하는 것이므로, 데이터 볼트 모델은 새틀라이트를 사용하여 행 데이터의 모든 변경 사항을 저장합니다. 따라서 새틀라이트의 데이터를 업데이트하거나 수정할 수 없습니다. 새틀라이트 로드 시 변경된 속성 값이 하나 이상 있는 소스 레코드만 새틀라이트에 추가됩니다. 

예를 들어, 초기 로드 전에 테이블 시트에 두 개의 레코드가 포함되어 있었다고 가정합니다. :
```
{('321', '1A', 'Business'),('321', '8A', 'Economy')}
```

초기 로딩은 위성 `s_seat`에 두 개의 레코드를 채웠습니다. :
```
{  
	('18354bfb73f4d63918a1ac4faf97264f', 2023-07-04 22:02:23, 'Business')  
	,('abcc466825f003fc395a67e4e23bcbf7', 2023-07-04 22:02:23, 'Economy')  
}
```

또한 테이블 좌석은 다음과 같이 일정 시간이 지난 후 업데이트되었다고 가정합니다. :
```
{('321', '1A', 'Business'), ('321', '8A', 'Business')}
```

즉 `('321', '8A')`로 식별된 레코드의  `fare_conditions` 값이 이코노미에서 비즈니스로 변경된 후 두번째 로딩으로 새틀라이트 `s_seat`가 다음과 같이 변경된 경우입니다. :
```
{  
	('18354bfb73f4d63918a1ac4faf97264f',  2023-07-04 22:02:23,  'Business')  
	,('abcc466825f003fc395a67e4e23bcbf7', 2023-07-04 22:02:23, 'Economy')  
	,('abcc466825f003fc395a67e4e23bcbf7', 2023-07-08 18:33:09, 'Business')  
}
```

즉, 두 번째 로딩 프로세스는 비즈니스 키 `('321','8A')`가 있는 좌석이 변경되었음을 식별합니다. 따라서 이 로드 프로세스는 새 로드 날짜로 새 버전의 레코드를 추가하고 이전 버전은 변경 없이 `s_seat`에 유지합니다. 허브 또는 링크에는 값의 변경 빈도 또는 레코드의 출처에 따라 속성을 그룹화하는 새틀라이트가 여러 개 있을 수 있습니다. 단순화를 위해 모델링에서 레코드 소스 속성은 언급하지 않습니다. 

새틀라이트는 다음과 같은 규칙을 따릅니다.:
- 새틀라이트에는 상위 테이블(허브 또는 링크)이 하나만 있습니다.
- 새틀라이트에는 외래 키가 포함되지 않습니다. 
- 새틀라이트의 기본 키는 상위 테이블의 해시 키와 로딩 날짜의 합성입니다. 
- 기존 레코드는 업데이트되거나 삭제되지 않습니다. 

## Modeling Transaction
항공편은 한 번만 발생할 수 있으므로 Figure1의 `flights` 테이블의 레코드는 업데이트되지 않습니다.(이미 발생했거나 취소된 항공편을 업데이트하는 것은 의미가 없습니다.) 이러한 종류의 레코드를 트랜잭션 또는 트랜잭션 데이터라고 합니다. 트랜잭션 데이터의 다른 예로는 센서 데이터, 날씨 데이터, 판매 데이터 등이 있습니다. 데이터 볼트 모델링에는 이러한 종류의 데이터를 모델링하는 두 가지 접근 방식, 즉 non-historized links(transactional links)와 hub-based transaction modeling이 있습니다. 

![2024-04-01-datavault-03.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-04-01-datavault-01.png?raw=true)

Figure3은 `Airlines` 스키마의 항공편에 대한 non-historized link 모델링을 보여줍니다. 이 접근 방식에서는 `flight_id` 자체를 허브가 있는 독립적인 비즈니스 키로 간주하지 않습니다. `flight_id`는 비즈니스에 의미가 없기 때문입니다. 이 키는 `flight` 테이블 내에서 항공편을 고유하게 식별하는 기능을 하는 대리 키입니다. 이 대리키는 Figure 2의 `l_fligth_airport_aircraft` 링크에 종속 자식키로 포함되어 있습니다. 이 링크는 항공기, 출발 공항 및 도착 공항이 동일하지만 적어도 출발 예정일이 다른 항공편을 구분하는 데 필요합니다. 따라서 해시 키의 계산에는 다음과 같은 종속 자식 키가 포함되어야 합니다. 

```SQL
lhk_flight_airport_aircraft =  
	md5(  
		flight_id::varchar  
		|| '|'  
		|| upper(trim(departure_airport))  
		|| '|'  
		|| upper(trim(arrival_airport))  
		|| '|'  
		|| upper(trim(aircraft_code))  
	)
```

비행의 컨텍스트는 Figure3의 새틀라이트 `s_flight`로 표시됩니다. 이 새틀라이트는 비행이 한 번만 발생할 수 있으므로 히스토리를 저장하지 않습니다. 

![2024-04-01-datavault-04.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-04-01-datavault-01.png?raw=true)

히스토리가 없는 링크 모델링과 달리 허브 기반 모델링은 `flight_id`를 독립적인 비즈니스 키로 간주하고 자체 새틀라이트를 가진 허브로 모델링하며 히스토리도 저장하지 않습니다. 허브 `h_flight`를 `Bookings` 스키마에서 항공편 테이블의 다른 비즈니스 키와 연결하기 위해 이 모델은 Figure4와 같이 `L_flight_airport_aircraft`링크를 도입합니다. 

이제 `tickets` 테이블 모델링 및  `fligths` 테이블과의 연관성과 관련하여 두 가지 모델링 접근 방식을 비교해보겠습니다. 두 접근 방식 모두 `tickets` 테이블의 모든 비즈니스 키(`ticket_no`)의 허브로 `h_ticket`을 모델링합니다. 그런 다음 허브 기반 트랜잭션 모델링에서는 Figure4에 표시된 대로 링크 `l_ticket_flight`를 생성하여 `h_ticket`을 허브 `h_flight`에 연결합니다. 그러나 비히스토리화 링크 접근 방식에서는 `h_ticket`을 위성 `s_flight`에 연결할 수 없습니다. `s_flight`에 도달할 수 있는 유일한 경로는 `l_ticket_airport_aircraft` 링크를 통한 경로뿐이며, 이를 위해서는 데이터 볼트 모델링에서 허용되는 `h_ticket`과 `l_flight_airport_aircraft`사이에 `l_ticket_flight_aircraft` 링크를 생성해야합니다. 그러나 이 모델링에서는 자식 링크인 `l_ticket_flight_airport_aircraft` 링크와 부모 링크인 `l_flight_airport_aircraft` 링크 간의 종속성을 생성하므로 후자의 링크가 변경되면 전자의 링크(자식 링크)가 변경되므로 `tickets` 테이블의 허브 기반 거래 모델링에 비해 비역사화 링크 접근법의 단점이 있습니다. 


## Reference  
[practical introduction to data vault modeling](https://medium.com/@nuhad.shaabani/practical-introduction-to-data-vault-modeling-1c7fdf5b9014)