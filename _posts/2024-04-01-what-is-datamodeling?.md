---
layout: post
title: "[Data Engineering] 데이터 모델링이란?(인몬, 킴볼, 데이터볼트)"
subtitle:
categories: DataEngineering
tags: []
---

## 데이터 웨어하우징 모델링이란? 
데이터 모델은 데이터가 실제 세계와 연관되는 방식을 나타냅니다. 데이터 모델링은 데이터의 일관성 있는 구조를 의도적으로 선택하는 작업이며, 데이터를 비즈니스에 유용하게 만드는 단계입니다. 

**정규화 기법**은 RDBMS 초기부터 데이터 모델링에 사용되어왔습니다. 지금도 여전히 운영 데이터베이스의 모델링은 정규화기법을 사용하며 **데이터 웨어하우징 모델링** 기법도 1990년대 초반부터 사용되어 온 것으로 보입니다. 데이터 웨어하우징 모델링은 분석 쿼리를 운영 데이터베이스에서 분리하는 것에서 시작되었습니다. 데이터 관리(특히 데이터 거버넌스 및 데이터 품질)의 인기가 높아짐에 따라 일관성 있는 비즈니스 로직의 필요성이 커지고 있고 데이터 웨어하우징(및 비즈니스 인텔리전스)은 아래와 같은 목표를 기반으로 발전해왔습니다. 

- DW/BI 시스템은 정보에 쉽게 액세스할 수 있어야 합니다. DW/BI 시스템의 내용을 이해할 수 있어야 합니다. 데이터는 개발자뿐만 아니라 **비즈니스 사용자도 직관적이고 명확하게 이해**할 수 있어야 합니다. 데이터의 구조와 레이블은 비즈니스 사용자의 사고 과정과 어휘를 모방해야 합니다.
- DW/BI 시스템은 정보를 일관되게 제공해야 합니다. DW/BI 시스템의 데이터는 신뢰할 수 있어야 합니다. 데이터는 다양한 소스에서 신중하게 수집되고, 정제되고, 품질이 보장되어야 하며, 사용자가 사용하기에 적합한 경우에만 공개되어야 합니다. 또한 일관성이란 데이터 원본 간에 DW/BI 시스템의 콘텐츠에 대한 공통된 레이블과 정의가 사용됨을 의미합니다. **두 성능 측정값의 이름이 같으면 같은 의미**여야 합니다. 반대로, 두 측정값이 같은 의미가 아니라면 서로 다른 레이블을 지정해야 합니다.
- DW/BI 시스템은 변화에 적응해야 합니다. 사용자 요구, 비즈니스 조건, 데이터, 기술은 모두 변화할 수 있습니다. DW/BI 시스템은 기존 데이터나 응용 프로그램을 무효화하지 않도록 이러한 불가피한 변화를 원활하게 처리하 도록 설계되어야 합니다.
- DW/BI 시스템은 적시에 정보를 제공해야 합니다. 운영 의사 결정에 DW/BI 시스템이 더욱 집중적으로 사용됨에 따라 원시 데이터를 몇 시간, 몇 분, 심 지어 몇 초 내에 실행 가능한 정보로 변환해야 할 수도 있습니다.

위 목표를 가지고 발전해온 대표적인 데이터 웨어하우스 모델링 방식들을 살펴보며 각 모델링 방법론이 어떻게 데이터의 일관성있는 구조를 만들어가는지 알아봅시다. 


## 대표적인 데이터 웨어하우스의 모델링
데이터 웨어하우스라는 개념이 생긴 이후 주목받아왔던 데이터 웨어하우스 모델링 기술인 인먼 방식, 킴벌 방식, 데이터 볼트 방식을 소개합니다. 이 모델링 방식들은 배타적인 것이 아니며, 기술을 결합해서 사용할 수 있습니다. 예를 들어, 데이터 볼트를 구축한 후 킴벌의 스타스키마를 추가하여 사용할 수 있습니다. 

### 인먼
데이터 웨어하우스의 아버지인 빌 인먼은 데이터 모델링에 대한 접근 방식을 고안했습니다. 데이터 웨어하우스 이전에는 원천 시스템에 직접 쿼리하여 분석을 하곤 했습니다. 장시간 실행되는 분석 쿼리로 인해 운영 트랜잭션 데이터베이스에 부하가 커졌고 시스템에 영향을 미치는 일이 생기기 시작했고 인먼은 원천 시스템과 분석 시스템을 분리하기 위해 데이터 웨어하우스를 정의했습니다. 인먼은 다음과 같이 데이터 웨어하우스를 정의합니다. 

```
데이터 웨어하우스는 경영진의 의사결정을 지원하기 위한 주제 지향적이고, 통합되고, 비휘발성이면서 시간 변이성을 가진 데이터 집합이다. 데이터 웨어하우스에는 세분화된 기업 데이터가 포함되어 있다. 데이터 웨어하우스의 데이터는 현재로서는 알 수 없는 미래의 요구 사항에 대기하는 등 다양한 용도로 사용할 수 있다. 
```

위 정의에 나타난 데이터 웨어하우스의 중요한 부분을 네 부분으로 쪼개어 봅시다. 
- 주제 지향성: 데이터 웨어하우스는 영업이나 마케팅 같은 특정 주제에 초점을 맞춘다. 
- 통합: 서로 다른 소스의 데이터는 통합되고 정규화된다. 
- 비휘발성: 데이터는 데이터 웨어하우스에 저장된 후에도 변경되지 않는다. 
- 시간 변이성: 다양한 시간 범위를 쿼리할 수 있다. 

![2024-04-01-inmon-dw.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-04-01-inmon-dw.png?raw=true)

인먼 방식은 엔터프라이즈 데이터 웨어하우스(EDW)를 중심으로 데이터 모델을 설계하며, 주로 정규화를 통해 데이터의 중복을 줄이고 일관성을 높입니다. 이는 기업 전체의 데이터를 통합하고, 중앙 집중화된 데이터 웨어하우스를 구축하는 데 중점을 둡니다. 

### 킴벌
킴벌의 데이터 모델링 접근 방식은 정규화에 초점을 맞추지 않으며 필요에 따라 비정규화를 수용합니다. 인먼 모델은 기업 전체 데이터를 데이터 웨어하우스에 통합하고 데이터 마트를 통해 부서별 분석을 제공하는 반면, 킴벌 모델은 상향식으로 데이터 웨어하우스 자체에서 부서 또는 비즈니스 분석을 모델링하고 제공하도록 권장합니다. 차원 모델링이라고도 부르는 킴벌의 모델링은 **비즈니스 사용자가 이해할 수 있는 데이터 제공**과 **빠른 쿼리 성능 제공**을 해결합니다. 차원 모델링은 데이터베이스를 단순하게 만들기 위한 기술입니다. 3NF(제 3정규형) 구조는 운영 처리에 매우 유용하지만 BI 쿼리에는 너무 복잡하기 때문입니다. 

킴벌의 접근법에서 데이터는 팩트 테이블과 차원 테이블이라는 두 가지 일반적인 유형의 테이블로 모델링됩니다. 팩트 테이블은 숫자 테이블, 차원 테이블은 팩트를 참조하는 정성적 데이터로 생각할 수 있습니다. 

#### 팩트 테이블
차원 모델링의 팩트 테이블에는 조직의 비즈니스 프로세스 이벤트에서 발생한 성과 측정값이 저장됩니다. 비즈니스 프로세스에서 발생하는 낮은 수준의 측정 데이터는 단일 차원 모델에 저장하도록 노력해야하고 측정 데이터는 엄청 큰 데이터 집합이므로 여러 곳에 복제해서는 안됩니다. 

**"Fact"** 는 비즈니스 척도를 나타냅니다. 마켓에서 제품을 관찰하며 각 제품의 단위 수량과 판매 금액을 기록한다고 할 때, *언제, 어디서, 누가, 무엇을, 얼마너치(id로 기록, 설명은 dim table에서)* 샀는지 등을 기록하는 것을 말합니다. 각 행의 데이터는 **"grain"** 이라는 세부 수준을 가집니다.(위에서는 grain이 제품 1개) grain은 얼마의 단위로 데이터를 저장하는지를 말합니다. 

#### 차원테이블
차원 테이블에는 비즈니스 프로세스 측정 이벤트와 관련된 텍스트 컨텍스트가 포함되어있습니다. 차원 테이블과 팩트 테이블을 결합하여 '누가 무엇을 어디서 언제 어떻게 왜'를 설명할 수 있습니다. 차원 테이블에 50-100개의 속성이 있는 것은 드문 일이 아닙니다. 차원 속성은 쿼리 제약 조건, 그룹화 및 보고서 레이블의 기본 소스 역할을 합니다. 

#### 스타스키마
스타스키마는 비즈니스의 데이터 모델을 나타냅니다. 고도로 정규화된 데이터 모델링 접근 방식과 달리 스타스키마는 필요한 차원으로 둘러 싸인 팩트 테이블입니다. 다른 데이터 모델보다 조인 수가 적어 쿼리 성능이 향상됩니다. 스타스키마는 비즈니스 로직의 팩트와 속성을 포착하고 각각의 중요한 질문에 답변할 수 있을 만큼 유연해야합니다. 
![2024-04-01-star_schema.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-04-01-star_schema.png?raw=true)

### 데이터 볼트
데이터 볼트 방법론은 댄 린스테드가 1990년대에 개발했습니다. 이 방법론은 원천 시스템 데이터의 속성에서 구조적 측면을 분리합니다. 비즈니스 로직을 팩트, 차원(킴벌) 또는 고도로 정규화된 테이블(인먼)로 표현하는 대신 원천 시스템의 데이터를 입력 전용 방식으로 소수의 특수 제작된 테이블에 직접 로드합니다(hub). 데이터 볼트 방법론의 목표는 "데이터 모델이 민첩하고 유연하며 확장 가능하도록"하는 것입니다. 많이 사용되던 킴벌의 차원 모델은 원천 데이터의 변화에 대응하기 어려웠습니다. 데이터 볼트는 새로운 소스와 데이터 구조를 추가할 때 재작업을 최소화합니다. 

데이터 볼트 모델은 허브, 링크, 위성이라는 세 가지 주요 유형의 테이블로 구성됩니다. 허브는 비즈니스 키를 저장하고, 링크는 비즈니스 키 간의 관계를 유지하며, 새틀라이트(위성)은 비즈니스 키의 속성과 컨텍스트를 표현합니다. 이제 허브, 링크, 새틀라이트를 더 자세히 살펴봅시다. 

#### 허브
쿼리에는 종종 고객 ID나 주문 ID와 같은 비즈니스 키를 통한 검색이 포함됩니다. 허브는 데이터 볼트에 로드된 모든 고유한 비즈니스 키의 레코드를 보관하는 엔티티입니다. 따라서 허브를 설계할 때는 비즈니스 키를 식별하는 것이 중요하고 비즈니스 키가 저장되는 허브는 영구히 변경되지 않아야합니다. 

#### 링크
링크 테이블은 허브 간 비즈니스 키의 관계를 추적합니다. 링크 테이블은 다양한 허브의 데이터를 연결하므로 다대다 관계입니다. 새로운 비즈니스 개념(새로운 허브)이 생성된다고 하더라도 기존의 테이블을 수정하는 것이 아니라 새로운 링크를 작성하므로써 관계를 표현할 수 있으므로 변화에 유연하게 대처할 수 있습니다. 

#### 새틀라이트(위성)
새틀라이트는 허브와 링크에 의미와 맥락을 부여합니다. 새틀라이트의 필수 필드는 부모 허브의 비스니스 키와 로드날짜 뿐입니다. 해시와 비즈니스 키, 타임스템프 등으로 구성된 허브, 링크에 유의미한 추가 정보들을 연결합니다. 


데이터 볼트에서는 위 세 테이블을 쿼리할 때 비즈니스 로직이 생성되고 해석됩니다. 데이터 볼트는 다른 데이터 모델링 기법과 함께 사용되는 경우가 많으며, 이후에 일반적으로 스타 스키마를 이용해 데이터 웨어하우스에서 별도로 모델링됩니다. 

## 결론